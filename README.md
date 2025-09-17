Criando um Microsserviço Serverless para Validação de CPF

A seguir, apresento um guia para criar um microsserviço serverless, usando por exemplo o Azure Functions com JavaScript, para realizar a validação de um CPF. O serviço receberá o CPF por meio de um endpoint HTTP e aplicará a lógica de validação (eliminação de caracteres não numéricos, verificação de dígitos iguais e cálculo dos dígitos verificadores).

Passos para criar o microsserviço
Criação da Function App: No portal do Azure, crie uma Function App com o runtime Node.js.. Esta será a base para hospedar o seu microserviço serverless.

Criação da Azure Function com disparo HTTP: Crie uma nova função do tipo HTTP trigger. Essa função receberá as requisições que conterão o CPF a ser validado.

Implementação da Lógica de Validação de CPF: A lógica de validação inclui:

Remover caracteres não numéricos;

Verificar se o CPF possui 11 dígitos;

Checar se todos os dígitos não são iguais (por exemplo, "11111111111" é inválido);

Calcular os dígitos verificadores (décimo e décimo primeiro dígitos) conforme o algoritmo brasileiro e compará-los com os informados.

Resposta da API: Caso o CPF seja válido, retorne uma resposta de sucesso; caso contrário, informe ao consumidor que o CPF é inválido.

Exemplo de Código (index.js)
javascript
module.exports = async function (context, req) {
    // Obtém o CPF enviado via query ou no corpo da requisição
    const cpf = req.query.cpf || (req.body && req.body.cpf);

    if (!cpf) {
        context.res = {
            status: 400,
            body: "Por favor, informe o CPF."
        };
        return;
    }

    // Remove caracteres não numéricos
    let cleanedCPF = cpf.replace(/\D/g, "");

    // Valida se o CPF possui 11 dígitos
    if (cleanedCPF.length !== 11) {
        context.res = {
            status: 400,
            body: "CPF deve conter 11 dígitos."
        };
        return;
    }

    // CPF com dígitos iguais não são válidos
    if (/^(\d)\1+$/.test(cleanedCPF)) {
        context.res = {
            status: 400,
            body: "CPF inválido."
        };
        return;
    }

    // Função para calcular o dígito verificador
    const calcularDigito = (cpfParte, pesoInicial) => {
        let soma = 0;
        for (let i = 0; i < cpfParte.length; i++) {
            soma += parseInt(cpfParte[i]) * (pesoInicial - i);
        }
        let resto = soma % 11;
        return resto < 2 ? 0 : 11 - resto;
    };

    // Calcula o primeiro dígito verificador usando os 9 primeiros dígitos
    const digito1 = calcularDigito(cleanedCPF.substr(0, 9), 10);
    if (digito1 !== parseInt(cleanedCPF[9])) {
        context.res = {
            status: 400,
            body: "CPF inválido."
        };
        return;
    }

    // Calcula o segundo dígito verificador usando os 10 primeiros dígitos
    const digito2 = calcularDigito(cleanedCPF.substr(0, 10), 11);
    if (digito2 !== parseInt(cleanedCPF[10])) {
        context.res = {
            status: 400,
            body: "CPF inválido."
        };
        return;
    }

    // Se passou por todas as validações, o CPF é válido
    context.res = {
        status: 200,
        body: "CPF válido."
    };
};
