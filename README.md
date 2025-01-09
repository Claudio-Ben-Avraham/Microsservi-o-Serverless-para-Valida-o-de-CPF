# Microsserviço Serverless para Validação de CPF

Este repositório contém a implementação de um microsserviço serverless para validação de CPF (Cadastro de Pessoa Física). Ele utiliza **AWS Lambda** para o processamento e **API Gateway** para expor uma API REST. O serviço valida se o CPF informado é válido ou inválido, considerando o cálculo dos **dígitos verificadores**.

## Arquitetura

- **AWS Lambda**: Função serverless que realiza a validação do CPF.
- **API Gateway**: Exposição da função Lambda como uma API RESTful.
- **CloudWatch Logs**: Para monitoramento e logs das execuções da função Lambda.

## Funcionalidade

A função Lambda recebe um **CPF** como parâmetro na URL, valida a formatação e os dígitos verificadores do CPF, e retorna uma resposta indicando se o CPF é válido ou inválido.

### Validação do CPF

O CPF é validado de acordo com as regras de dígitos verificadores:

- Um CPF válido possui 11 dígitos numéricos.
- Os dois últimos dígitos (dígitos verificadores) são calculados a partir dos primeiros 9 números.

## Tecnologias Usadas

- **AWS Lambda** (Node.js)
- **API Gateway**
- **AWS CloudWatch Logs**

## Como Criar o Microsserviço

### Passo 1: Criar a Função Lambda

1. Acesse o **AWS Management Console**.
2. Vá para o serviço **Lambda** e clique em **Create function**.
3. Escolha **Author from scratch** e configure a função:
   - Nome: **ValidarCPF**
   - Runtime: **Node.js**
   - Role: Selecione ou crie uma Role com permissões mínimas necessárias.
4. Cole o código da função Lambda abaixo.

### Código da Função Lambda

```javascript
exports.handler = async (event) => {
    const cpf = event.queryStringParameters.cpf;

    // Validação do CPF
    if (!cpf || cpf.length !== 11 || !/^\d+$/.test(cpf)) {
        return {
            statusCode: 400,
            body: JSON.stringify({ message: "CPF inválido. Deve conter 11 dígitos numéricos." }),
        };
    }

    // Função para validar os dois dígitos verificadores
    const validarCpf = (cpf) => {
        let soma1 = 0;
        let soma2 = 0;
        let multiplicador1 = 10;
        let multiplicador2 = 11;

        // Validação do primeiro dígito verificador
        for (let i = 0; i < 9; i++) {
            soma1 += parseInt(cpf[i]) * multiplicador1--;
        }
        const resto1 = (soma1 * 10) % 11;
        const digito1 = resto1 === 10 ? 0 : resto1;
        
        // Validação do segundo dígito verificador
        for (let i = 0; i < 9; i++) {
            soma2 += parseInt(cpf[i]) * multiplicador2--;
        }
        soma2 += digito1 * 2;
        const resto2 = (soma2 * 10) % 11;
        const digito2 = resto2 === 10 ? 0 : resto2;

        return cpf[9] == digito1 && cpf[10] == digito2;
    };

    if (validarCpf(cpf)) {
        return {
            statusCode: 200,
            body: JSON.stringify({ message: "CPF válido." }),
        };
    } else {
        return {
            statusCode: 400,
            body: JSON.stringify({ message: "CPF inválido." }),
        };
    }
};
## Passo 2: Criar a API REST com API Gateway

1. No **AWS Management Console**, vá para **API Gateway**.
2. Crie uma nova **API REST**.
   - Nome da API: `CPFValidationAPI`.
3. Adicione um novo recurso: **/validate-cpf**.
4. Crie o método **GET** e associe-o à função Lambda **ValidarCPF**.
   - Para isso, selecione o método GET e configure a integração com a função Lambda.
5. Configure o método GET para invocar a função Lambda **ValidarCPF**.
   - Após configurar, clique em **Deploy API**.
6. Escolha um novo **Stage**, por exemplo, `prod` e clique em **Deploy**.

Após a implantação, o API Gateway fornecerá uma URL pública para a API, que você pode usar para fazer requisições.

---

## Passo 3: Configuração de Segurança

A configuração de segurança pode ser feita de diferentes maneiras. Você pode escolher uma abordagem mais simples ou mais complexa, dependendo do seu caso de uso.

1. **Sem autenticação**: Deixe a API sem autenticação para acesso público.
2. **API Key**: Configure a autenticação via API Key para controlar o acesso.
3. **Autenticação com IAM roles**: Configure permissões usando IAM roles para proteger a API.

Escolha a opção que melhor se adapta às suas necessidades.

---

## Passo 4: Implantação da API

1. Após configurar e testar o método, implemente a API em um novo **Stage**. Um stage pode ser `dev`, `test` ou `prod` conforme o ambiente de uso.
2. O **API Gateway** fornecerá uma URL pública para sua API, como a seguinte:


Essa URL é o endpoint que você usará para fazer as requisições de validação de CPF.

---

## Testando a API

Depois de implantar a API, você pode testá-la facilmente através de um **GET** para a URL fornecida pelo **API Gateway**. 

### Exemplo de requisição:


### Respostas da API:

- Se o CPF for válido, a resposta será:

```json
{
   "message": "CPF válido."
}
{
   "message": "CPF inválido."
}
Monitoramento e Logs

    Ative o CloudWatch Logs para a função Lambda para monitorar a execução e verificar se houve algum erro durante o processamento do CPF.
    Para isso, acesse o AWS Lambda, selecione a função ValidarCPF e ative o CloudWatch Logs.
    Você pode monitorar as execuções da função Lambda e acompanhar os logs gerados, o que é útil para depuração e análise de performance.

Conclusão

Este microsserviço serverless é uma solução escalável e eficiente para validação de CPF. Ele utiliza AWS Lambda para o processamento e API Gateway para expor a API de forma segura e prática. O serviço é ideal para validação automatizada de CPF em qualquer aplicação que precise dessa funcionalidade.

Com a utilização do AWS CloudWatch, é possível acompanhar o desempenho e eventuais erros, tornando o serviço fácil de monitorar e ajustar conforme a necessidade. A API pode ser facilmente consumida por outros sistemas ou usuários, oferecendo uma solução ágil e eficiente.


---

Esses passos detalham as ações a serem tomadas no **AWS Management Console**, explicando como configurar, testar, monitorar e finalizar o processo de implementação do microsserviço serverless. Você pode copiar e colar esse conteúdo diretamente no seu **README.md**!
