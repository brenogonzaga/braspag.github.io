---
layout: manual
title: Manual de Integração - Antifraude
description: Integração Técnica API Antifraude Gateway Braspag
search: true
translated: true
categories: manual
tags:
  - 4. Gestão de Risco
language_tabs:
  json: JSON
  
---

# Sobre essa documentação 

Este manual apresenta a integração com a API do Antifraude Gateway Braspag para chamadas apartadas de análise de fraude. Para integração de chamada conjunta com outros serviços, veja o manual do serviço de interesse: [Pagador](https://braspag.github.io//manual/braspag-pagador#pagamentos-com-an%C3%A1lise-de-fraude){:target="_blank"}, [API Cielo E-commerce](https://braspag.github.io//manual/api-ecommerce-cielo-af){:target="_blank"}, [Split Cielo E-commerce](https://braspag.github.io//manual/split-de-pagamentos-cielo-e-commerce#antifraude){:target="_blank"} ou [Split Pagador](https://braspag.github.io//manual/split-de-pagamentos-pagador#antifraude){:target="_blank"}.

# Visão Geral

O Antifraude Gateway da Braspag é uma API REST que permite a gestão de risco do seu e-commerce pela análise dos melhores provedores de antifraude do mundo.

## Por que ter uma análise de fraude?

* Consiga a melhor conversão controlando o índice de chargeback;
* Identifique transações fraudulentas;
* Aumente a segurança pela análise de histórico  de transações do Antifraude;
* Evite multas das bandeiras por aumento no índice de chargeback.

## Benefícios do Antifraude Gateway

O Antifraude Gateway Braspag é um facilitador do processo de análise de fraude do seu e-commerce, permitindo:

* Acesso aos melhores provedores de antifraude do mercado, sem o ônus de uma integração direta; 
* Integração simplificada;
* Facilidade: as principais plataformas de e-commerce já estão integradas à Braspag.

## Provedores de antifraude

Os provedores de antifraude oferecem ferramentas baseadas em rede neural, bancos de dados não relacionais, recomendações baseadas em histórico de transações, comportamento e repetições por intervalo de tempo, entre outras funções que compõem o modelo de análise de fraude.

Na contratação do Antifraude Gateway, você escolhe com qual(is) provedor(es) deseja analisar suas transações e o plano. 

* **Cybersource**: é uma empresa do grupo Visa, que atua desde 1994 com gerenciamento de fraude. Os planos da Cybersource são:
> * **Básico** (antigo Hierarquia): modelos de risco padronizados pelo segmento de mercado do e-commerce;
> * **Avançado** (antigo Advanced): permite proposição de melhorias nas regras de risco e consultoria mensal com a Cybersource e Braspag;
> * **Completo** (antigo Enterprise): conta com um analista de risco dedicado da CyberSource, que irá propor proativamente ajustes no conjunto de regras do negócio a fim de melhorar a conversão e reduzir índices de chargeback.

* **ACI Worldwide**: é uma empresa global de software de missão crítica que fornece soluções de controle e prevenção à fraude bem como gestão e processamento de pagamentos em tempo real para corporações.
* **ClearSale**: é autoridade em soluções antifraude, equilibrando tecnologia avançada e inteligência humana especializada para entregar excelentes indicadores e gerar confiança.

> Neste manual, consulte as requisições e [tabelas anexas correspondentes ao seu provedor de antifraude](https://braspag.github.io//manual/antifraude#tabelas).

# Fluxos da Análise de Fraude

O Antifraude Gateway encaminha os dados de uma transação para análise de um provedor de antifraude. Os provedores utilizam tecnologias como identificação de máquina, geolocalização de IP, análise de redes sociais, detecção de proxy e contadores de repetição. A Braspag recebe uma recomendação do provedor em tempo real podendo então tomar uma ação baseada no risco de fraude identificado pela análise.
Existem dois fluxos possíveis para a análise de fraude:

* **AnalyseFirst:** todas as transações são primeiro submetidas à análise do Antifraude e, se aceitas, seguem para o processo de autorização da adquirente. 
* **AuthorizeFirst:** todas as transações são primeiro submetidas à autorização na adquirente e, se autorizadas, são enviadas para a análise do Antifraude. 

![Fluxos AnalyzeFirst e AuthorizeFirst]({{ site.baseurl_root }}/images/braspag/af/fluxo-authorizefirst-analyzefirst100.png)

*O status Review é retornado somente se estiver incluso no seu plano.

## AnalyzeFirst

1. A loja solicita a análise de fraude ao Antifraude;
2. O Antifraude retorna o status Accept, Reject ou Review;
3. A loja envia a requisição da transação para o gateway de pagamentos;
4. O gateway de pagamentos solicita autorização da adquirente;
5. A adquirente autoriza a transação ou autoriza e captura;
6. O gateway de pagamentos retorna a resposta da autorização.

## AuthorizeFirst

1. A loja envia a requisição da transação para o gateway de pagamentos;
2. O gateway de pagamentos solicita autorização da adquirente;
3. A adquirente autoriza a transação;
4. O gateway de pagamentos retorna a resposta da autorização;
5. A loja solicita a análise de fraude ao Antifraude;
6. O Antifraude retorna o status Accept, Reject ou Review.

O fluxo escolhido deve ser determinado durante a integração, porque faz parte da arquitetura da aplicação.

Se após a integração você desejar alterar o fluxo de análise de fraude solicite o suporte da Braspag, pois existem detalhes envolvidos em uma possível troca, principalmente em relação às regras de velocidade.

## Qual fluxo é indicado para o meu e-commerce?

* A escolha pode ser feita de acordo com as regras do seu negócio, levando em conta aspectos como o ticket médio e tipo de produto comercializado;
* Você pode montar a sua aplicação com os dois fluxos e aplicar as regras de negócio de acordo com seus critérios de risco.

Confira a seguir algumas características de cada fluxo, que podem auxiliar você na decisão do melhor modelo para o seu negócio: 

**AnalyzeFirst**

* O Antifraude terá visibilidade das transações de cartão de crédito do e-commerce;
* Evita sensibilização antecipada do limite do cartão de crédito, evitando atrito/má experiência do consumidor;
* Favorece o aprendizado do motor do Antifraude;
* Há custo de análise quando o cartão de crédito usado na transação não tem limite disponível.

**AuthorizeFirst**

* Evita o custo da análise em transações de cartão de crédito que não tem limite disponível;
* O Antifraude não tem visibilidade das transações não autorizadas.

# Ambientes

## Sandbox

|Ambiente|URL base|Descrição|
|:-|:-|:-|
|**Braspag OAUTH2 Server**|https://authsandbox.braspag.com.br/ | Autenticação.|
|**API Risk**|https://risksandbox.braspag.com.br/ | Análise de fraude, consulta, associar transação e update de status. |

## Produção

|Ambiente|URL base|Descrição|
|:-|:-|:-|
|**Braspag OAUTH2 Server**|https://auth.braspag.com.br/ | Autenticação.|
|**API Risk**|https://risk.braspag.com.br/ | Análise de fraude, consulta, associar transação e update de status. |

# Autenticação

A API Antifraude Gateway Braspag utiliza o protocolo padrão de mercado OAuth 2.0 para autorização de acesso a seus recursos específicos por ambientes, **Sandbox** e **Produção**.

## Como obter o token de acesso

Durante o onboarding, você receberá as credenciais `ClientId` e `ClientSecret`. Caso não tenha recebido a credencial, solicite ao [Suporte Braspag](https://suporte.braspag.com.br/hc/pt-br){:target="_blank"}.

**1.** Concatene as credenciais no formato `ClientId:ClientSecret`;<br/>
**2.** Converta o resultado em base 64, gerando uma string;

> **Exemplo:**<br/>
> * client_id: **braspagtestes**<br/>
> * client_secret: **1q2w3e4r5t6y7u8i9o0p0q9w8e7r6t5y4u3i2o1p**<br/>
> * String a ser codificada em Base64: **braspagtestes:1q2w3e4r5t6y7u8i9o0p0q9w8e7r6t5y4u3i2o1p**<br/>
> * Resultado após a codificação: **YnJhc3BhZ3Rlc3RlczoxcTJ3M2U0cg==**<br/>

**3.** Envie a string em base 64 na requisição de Autenticação (POST);<br/>
**4.** A API de Autenticação irá validar a string e retornará o `access_token` (token de acesso). 

O token retornado (`access_token`) deverá ser utilizado em toda requisição à API do Antifraude Gateway como uma chave de autorização. O `access_token` possui uma validade de 20 minutos e é necessário gerar um novo toda vez que a validade expirar.

Veja na imagem o esquema da autenticação e envio do `access_token` na requisição de análise de fraude.

![Fluxo de Autenticação]({{ site.baseurl_root }}/images/braspag/af/af-autenticacao-bpauth.png)

### Requisição

<aside class="request"><span class="method post">POST</span> <span class="endpoint">oauth2/token</span></aside>

**Parâmetros no Cabeçalho (header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/x-www-form-urlencoded|
|`Authorization`|Basic YnJhc3BhZ3Rlc3RlczoxcTJ3M2U0cg==|

**Parâmetros no Corpo (body)**

|Key|Value|
|:-|:-|
|`scope`|AntifraudGatewayApp|
|`grant_type`|client_credentials|

### Resposta

``` json
{
  "access_token": "faSYkjfiod8ddJxFTU3vti_ ... _xD0i0jqcw",
  "token_type": "bearer",
  "expires_in": 599
}
```

**Parâmetros no corpo (body)**

|Parâmetro|Descrição|
|:-|:-|
|`access_token`|O token de acesso solicitado.|
|`token_type`|Indica o valor do tipo de token.|
|`expires_in`|Expiração do token de acesso, em segundos. <br/>Após expirar, é necessário obter um novo.|

# Integração com a Cybersource

A seguir, apresentamos um exemplo de requisição de análise de fraude com a Cybersource.

> Atenção: Você só deve enviar o campo `BraspagTransactionId` se o seu fluxo for `AuthorizeFirst` e estiver usando o Pagador Braspag. O campo `BraspagTransactionId` é o identificador da transação no Pagador. Saiba mais na [documentação do Pagador](https://braspag.github.io//manual/braspag-pagador#pagamentos-com-an%C3%A1lise-de-fraude){:target="_blank"}.

## Analisando uma transação na Cybersource

### Requisição

<aside class="request"><span class="method post">POST</span> <span class="endpoint">analysis/v2/</span></aside>

``` json
{
  "MerchantOrderId": "4493d42c-8732-4b13-aadc-b07e89732c26",
  "TotalOrderAmount": 15000,
  "TransactionAmount": 14000,
  "Currency": "BRL",
  "Provider": "Cybersource",
  "BraspagTransactionId":"a3e08eb2-2144-4e41-85d4-61f1befc7a3b",
  "Tid": "12345678910111216AB8",
  "Nsu": "951852",
  "AuthorizationCode":"T12345",
  "SaleDate": "2016-12-09 10:01:55.662",
  "Card": {
    "Number" : "4444555566667777",
    "Holder": "Holder Name",
    "ExpirationDate": "12/2023",
    "Cvv": "999",
    "Brand": "VISA"
  },
  "Billing": {
    "Street": "Rua Saturno",
    "Number": "12345",
    "Complement": "Sala 123",
    "Neighborhood": "Centro",
    "City": "Rio de Janeiro",
    "State": "RJ",
    "Country": "BR",
    "ZipCode": "20080123"
  },
  "Shipping": {
    "Street": "Rua Neturno",
    "Number": "30000",
    "Complement": "sl 123",
    "Neighborhood": "Centro",
    "City": "Rio de Janeiro",
    "State": "RJ",
    "Country": "BR",
    "ZipCode": "123456789",
    "FirstName": "João",
    "LastName": "Silva",
    "ShippingMethod": "SameDay",
    "Phone": "552121114700"
  },
  "Customer": {
    "MerchantCustomerId": "10050665740",
    "FirstName": "João",
    "LastName": "Silva",
    "BirthDate": "2016-12-09",
    "Email": "emailcomprador@dominio.com.br",
    "Phone": "552121114700",
    "Ip": "127.0.0.1",
    "BrowserHostName":"www.dominiobrowsercomprador.com.br",
    "BrowserCookiesAccepted":true,
    "BrowserEmail":"emailbrowsercomprador@dominio.com.br",
    "BrowserType":"Chrome 58 on Windows 10",
    "BrowserFingerprint":"123456789"
  },
  "CartItems": [
    {
      "ProductName": "Mouse",
      "Category": "EletronicGood",
      "UnitPrice": "12000",
      "Sku": "abc123",
      "Quantity": 1,
      "Risk":"Low",
      "AddressRiskVerify":"No",
      "HostHedge":"Low",
      "NonSensicalHedge":"Normal",
      "ObscenitiesHedge":"High",
      "TimeHedge":"Low",
      "PhoneHedge":"Normal",
      "VelocityHedge":"High"
    },
    {
      "ProductName": "Teclado",
      "Category": "EletronicGood",
      "UnitPrice": "96385",
      "MerchantItemId": "3",
      "Sku": "abc456",
      "Quantity": 1,
      "Risk": "High"
    }
  ],
  "MerchantDefinedData": [
    {
      "Key": "1",
      "Value": "Valor definido com o Provedor a ser enviado neste campo."
    },
    {
      "Key": "2",
      "Value": "Valor definido com o Provedor a ser enviado neste campo."
    },
    {
      "Key": "3",
      "Value": "Valor definido com o Provedor a ser enviado neste campo."
    }
  ],
  "Bank":{
    "Address": "Rua Marte, 29",
    "Code": "237",
    "Agency": "987654",
    "City": "Rio de Janeiro",
    "Country": "BR",
    "Name": "Bradesco",
    "SwiftCode": "789"
  },
  "FundTransfer":{
    "AccountNumber":"159753",
    "AccountName":"Conta particular",
    "BankCheckDigit":"51",
    "Iban":"123456789"
  },
  "Invoice":{
    "IsGift": false,
    "ReturnsAccept": true,
    "Tender": "Consumer"
  },
  "Airline": {
    "JourneyType": "OneWayTrip",
    "DepartureDateTime": "2018-01-09 18:00",
    "Passengers": [
    {
        "FirstName": "Fulano",
        "LastName": "Tal",
        "PassangerId": "",
        "PassengerType": "Adult",
        "Email": "email@mail.com",
        "Phone": "1234567890",
        "Status": "Gold",
        "Legs" : [
        {
            "ArrivalAirport": "AMS",
            "DepartureAirport": "GIG"
        }]
    }]
  }
}
```

**Parâmetros no cabeçalho (Header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|
|`Authorization`|Bearer {access_token}|
|`MerchantId`|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|
|`RequestId`|nnnnnnnn-nnnn-nnnn-nnnn-nnnnnnnnnnnn|

**Parâmetros no corpo (body)**

|Parâmetro|Descrição|Tipo|Obrigatório|Tamanho|
|:-|:-|:-:|:-:|-:|
|`MerchantOrderId` |Número do pedido da loja|string|Sim|100|
|`TotalOrderAmount`|Valor total do pedido em centavos <br/> Ex: 123456 = r$ 1.234,56|long|Sim|-|
|`TransactionAmount`|Valor da transação financeira em centavos <br/> Ex: 150000 = r$ 1.500,00|long|Sim|-|
|`Currency`|Moeda. Maiores informações em [ISO 4217 Currency Codes](https://www.iso.org/iso-4217-currency-codes.html){:target="_blank"}|string|Sim|3|
|`Provider`|Provedor da solução de antifraude <br/> [Tabela 1 - Provider](https://braspag.github.io/manual/antifraude#tabela-1-provider)|enum|-|-|
|`BraspagTransactionId`|Id da transação no Pagador da Braspag. Obs.: Você pode enviar esse campo se o seu fluxo de análise de fraude é AuthorizeFirst, no qual a autorização acontece primeiro. Se você não tem integração com o Pagador Braspag, você pode enviar os campos `Tdi`,`Nsu`, `AuthorizationCode` e `SaleDate` em vez do campo `BraspagTransactionId`|GUID|Não|-|
|`Tid`|Id da transação na adquirente <br/> Obs.: Você pode enviar esse campo se o seu fluxo de análise de fraude é AuthorizeFirst, no qual a autorização acontece primeiro. Se você não tem integração com o Pagador Braspag, você tem a opção de enviar o `Tid` acompanhado dos campos `Nsu`, `AuthorizationCode` e `SaleDate` em vez do campo `BraspagTransactionId`|string|Não|20|
|`Nsu`|Número sequencial único da transação na adquirente <br/> Obs.: Você pode enviar esse campo se o seu fluxo de análise de fraude é AuthorizeFirst, no qual a autorização acontece primeiro. Se você não tem integração com o Pagador Braspag, você tem a opção de enviar o `Nsu` acompanhado dos campos `Tid`, `AuthorizationCode` e `SaleDate` em vez do campo `BraspagTransactionId`|string|Não|10|
|`AuthorizationCode`|Código de autorização da transação na adquirente <br/> Obs.: Você pode enviar esse campo se o seu fluxo de análise de fraude é AuthorizeFirst, no qual a autorização acontece primeiro. Se você não tem integração com o Pagador Braspag, você tem a opção de enviar o `AuthorizationCode`, acompanhado dos campos `Tid`, `Nsu` e `SaleDate` em vez do campo `BraspagTransactionId`|string|Não|10|
|`SaleDate`|Data da autorização da transação na adquirente <br/> Obs.: Você pode enviar esse campo se o seu fluxo de análise de fraude é AuthorizeFirst, no qual a autorização acontece primeiro. Se você não tem integração com o Pagador Braspag, você tem a opção de enviar este campo, acompanhado dos campos `Tid`, `Nsu` e `AuthorizationCode` em vez do campo `BraspagTransactionId`|datetime|Não|-|
|`Card.Number`|Número do cartão de crédito|string|Sim|19|
|`Card.Holder`|Nome do portador impresso no cartão de crédito|string|Sim|50|
|`Card.ExpirationDate`|Data de expiração do cartão de crédito <br/> Ex.: 01/2023|string|Sim|7|
|`Card.Brand`|Bandeira do cartão de crédito <br/> [Tabela 3 - Card.Brand](https://braspag.github.io/manual/antifraude#tabela-3-card.brand)|enum|Sim|-|
|`Card.Save`|Indica se os dados do cartão de crédito serão armazenados no Cartão Protegido|booleano|Não|-|
|`Card.Token`|Identificador do cartão de crédito salvo no Cartão Protegido|GUID|Não|-|
|`Card.Alias`|*Alias* (apelido) do cartão de crédito salvo no Cartão Protegido|string|Não|64|
|`Billing.Street`|Logradouro do endereço de cobrança|string|Sim|54|
|`Billing.Number`|Número do endereço de cobrança|string|Sim|5|
|`Billing.Complement`|Complemento do endereço de cobrança|string|Não|14|
|`Billing.Neighborhood`|Bairro do endereço de cobrança|string|Sim|45|
|`Billing.City`|Cidade do endereço de cobrança|string|Sim|50|
|`Billing.State`|Estado do endereço de cobrança|string|Sim|2|
|`Billing.Country`|País do endereço de cobrança. Mais informações em [ISO 2-Digit Alpha Country Code](https://www.iso.org/obp/ui){:target="_blank"}|string|Sim|2|
|`Billing.ZipCode`|Código postal do endereço de cobrança|string|Sim|9|
|`Shipping.Street`|Logradouro do endereço de entrega|string|Não|54|
|`Shipping.Number`|Número do endereço de entrega|string|Não|5|
|`Shipping.Complement`|Complemento do endereço de entrega|string|Não|14|
|`Shipping.Neighborhood`|Bairro do endereço de entrega|string|Não|45|
|`Shipping.City`|Cidade do endereço de entrega|string|Não|50|
|`Shipping.State`|Estado do endereço de entrega|string|Não|2|
|`Shipping.Country`|País do endereço de entrega. Mais informações em [ISO 2-Digit Alpha Country Code](https://www.iso.org/obp/ui){:target="_blank"}|string|Não|2|
|`Shipping.ZipCode`|Código postal do endereço de entrega|string|Não|9|
|`Shipping.FirstName`|Primeiro nome do responsável a receber o produto no endereço de entrega|string|Não|60|
|`Shipping.LastName`|Último do nome do responsável a receber o produto no endereço de entrega|string|Não|60|
|`Shipping.Phone`|Número do telefone do responsável a receber o produto no endereço de entrega <br/> Ex.: 552121114700|string|Não|15|
|`Shipping.ShippingMethod`|Meio de entrega do pedido <br/> [Tabela 4 - ShippingMethod](https://braspag.github.io/manual/antifraude#tabela-4-shippingmethod)|enum|-|-|
|`Customer.MerchantCustomerId`|Número do documento de identificação do comprador, CPF ou CNPJ|string|Sim|16|
|`Customer.FirstName`|Primeiro nome do comprador|string|Sim|60|
|`Customer.LastName`|Último nome do comprador|string|Sim|60|
|`Customer.BirthDate`|Data de nascimento do comprador <br/> Ex.: 1983-10-01|date|Sim|-|
|`Customer.Email`|E-mail do comprador|string|Sim|100|
|`Customer.Ip`|Endereço de IP do comprador. Formato IPv4 ou IPv6|string|Sim|45|
|`Customer.Phone`|Número do telefone do comprador <br/> Ex.: 552121114700|string|Sim|15|
|`Customer.BrowserHostName`|Nome do host informado pelo browser do comprador e identificado através do cabeçalho HTTP|string|Não|60|
|`Customer.BrowserCookiesAccepted`|Identifica se o browser do comprador aceita cookies <br/> Possíveis valores: true / false (default)|bool|-|-|
|`Customer.BrowserEmail`|E-mail registrado no browser do comprador. Pode diferenciar do e-mail de cadastro na loja (`Customer.Email`)|string|Não|100|
|`Customer.BrowserType`|Nome do browser utilizado pelo comprador e identificado através do cabeçalho HTTP <br/> Ex.: Google Chrome, Mozilla Firefox, Safari etc.|string|Não|40|
|`Customer.BrowserFingerprint`|É o valor do `ProviderIdentifier`. Identificador utilizado para cruzar informações obtidas do dispositivo do comprador.<br>Obs.: Este identificador poderá ser qualquer valor ou o número do pedido, mas deverá ser único durante 48 horas.<br>Saiba mais em [Fingerprint com a Cybersource](https://braspag.github.io//manual/antifraude#fingerprint-com-a-cybersource)|string|Sim|88|
|`CartItem[n].ProductName`|Nome do produto|string|Sim|255|
|`CartItem[n].Category`|Categoria do produto <br/> [Tabela 36 - CartItem{n}.Category](https://braspag.github.io/manual/antifraude#tabela-36-cartitem[n].category)|enum|-|-|
|`CartItem[n].Risk`|Nível de risco do produto associado a quantidade de chargebacks <br/> [Tabela 10 - CartItem{n}.Risk](https://braspag.github.io/manual/antifraude#tabela-10-cartitem[n].risk)|enum|-|-|
|`CartItem[n].UnitPrice`|Preço unitário do produto <br/> Ex: 10950 = r$ 109,50|long|Sim|-|
|`CartItem[n].Sku`|SKU (Stock Keeping Unit - Unidade de Controle de Estoque) do produto|string|Sim|255|
|`CartItem[n].Quantity`|Quantidade do produto|int|Sim|-|
|`CartItem[n].AddressRiskVerify`|Identifica que avaliará os endereços de cobrança e entrega para diferentes cidades, estados ou países <br/> [Tabela 11 - CartItem{n}.AddressRiskVerify](https://braspag.github.io/manual/antifraude#tabela-11-cartitem[n].addressriskverify)|enum|-|-|
|`CartItem[n].HostHedge`|Nível de importância dos endereços de IP e e-mail do comprador na análise de fraude <br/> [Tabela 12 - CartItem{n}.HostHedge](https://braspag.github.io/manual/antifraude#tabela-12-cartitem[n].hosthedge)|enum|-|-|
|`CartItem[n].NonSensicalHedge`|Nível de importância das verificações sobre os dados do comprador sem sentido na análise de fraude <br/> [Tabela 13 - CartItem{n}.NonSensicalHedge](https://braspag.github.io/manual/antifraude#tabela-13-cartitem[n].nonsensicalhedge)|enum|-|-|
|`CartItem[n].ObscenitiesHedge`|Nível de importância das verificações sobre os dados do comprador com obscenidade na análise de fraude <br/> [Tabela 14 - CartItem{n}.ObscenitiesHedge](https://braspag.github.io/manual/antifraude#tabela-14-cartitem[n].obscenitieshedge)|enum|-|-|
|`CartItem[n].TimeHedge`|Nível de importância da hora do dia na análise de fraude que o comprador realizou o pedido <br/> [Tabela 15 - CartItem{n}.TimeHedge](https://braspag.github.io/manual/antifraude#tabela-15-cartitem[n].timehedge)|enum|-|-|
|`CartItem[n].PhoneHedge`|Nível de importância das verificações sobre os números de telefones do comprador na análise de fraude <br/> [Tabela 16 - CartItem{n}.PhoneHedge](https://braspag.github.io/manual/antifraude#tabela-16-cartitem[n].phonehedge)|enum|-|-|
|`CartItem[n].VelocityHedge`|Nível de importância da frequência de compra do comprador na análise de fraude dentro dos 15 minutos anteriores <br/> [Tabela 17 - CartItem{n}.VelocityHedge](https://braspag.github.io/manual/antifraude#tabela-17-cartitem[n].velocityhedge){:target="_blank"}|enum|-|-|
|`Bank.Name`|Nome do banco do comprador|string|Não|40|
|`Bank.Code`|Código do banco do comprador|string|Não|15|
|`Bank.Agency`|Agência do banco do comprador|string|Não|15|
|`Bank.Address`|Endereço do banco do comprador|string|Não|255|
|`Bank.City`|Cidade onde está localizado o banco do comprador|string|Não|15|
|`Bank.Country`|País onde está localizado o banco do comprador <br/> Mais informações em [ISO 2-Digit Alpha Country Code](https://www.iso.org/obp/ui){:target="_blank"}|string|não|2|
|`Bank.SwiftCode`|Código identificador único do banco do comprador|string|Não|30|
|`FundTransfer.AccountName`|Nome vinculado a conta bancária|string|Não|30|
|`FundTransfer.AccountNumber`|Número da conta bancária do comprador|string|Não|30|
|`FundTransfer.BankCheckDigit`|Código utilizado para validar a conta bancária do comprador|string|Não|2|
|`FundTransfer.Iban`|Número internacional da conta bancária do comprador (IBAN)|string|Não|30|
|`Invoice.IsGift`|Indica se o pedido realizado pelo comprador é para presente|booleano|Não|-|
|`Invoice.ReturnsAccepted`|Indica se o pedido realizado pelo comprador pode ser devolvido a loja|booleano|Não|-|
|`Invoice.Tender`|Forma de pagamento utilizada pelo comprador <br/> [Tabela 18 - Invoice.Tender](https://braspag.github.io/manual/antifraude#tabela-18-invoice.tender)|enum|Não|-|
|`Airline.JourneyType`|Tipo de viagem <br/> [Tabela 7 - Airline.JourneyType](https://braspag.github.io/manual/antifraude#tabela-7-airline.journeytype)|enun|Não|-|
|`Airline.DepartureDateTime`|Data e hora de partida <br/> Ex.: 2018-03-31 19:16:38|datetime|Não|-|
|`Airline.Passengers[n].FirstName`|Primeiro nome do passageiro|string|Não|60|
|`Airline.Passengers[n].LastName`|Último nome do passageiro|string|Não|60|
|`Airline.Passengers[n].PassengerId`|Identificador do passageiro a quem a passagem foi emitida|string|Não|32|
|`Airline.Passengers[n].PassengerType`|Tipo do passageiro <br/> [Tabela 8 - Airline.Passengers{n}.PassengerType](https://braspag.github.io/manual/antifraude#tabela-8-airline.passengers[n].passengertype)|enum|Não|-|
|`Airline.Passengers[n].Phone`|Telefone do passageiro <br/> Ex.: 552121114700|string|Não|15|
|`Airline.Passengers[n].Email`|E-mail do passageiro|string|Não|255|
|`Airline.Passengers[n].Status`|Classificação da empresa aérea <br/> [Tabela 9 - Airline.Passengers{n}.Status](https://braspag.github.io/manual/antifraude#tabela-9-airline.passengers[n].status)|enum|Não|60|
|`Airline.Passengers[n].Legs[n].DepartureAirport`|Código do aeroporto de partida. Mais informações em [IATA 3-Letter Codes](http://www.nationsonline.org/oneworld/IATA_Codes/airport_code_list.htm){:target="_blank"}|string|Não|3|
|`Airline.Passengers[n].Legs[n].ArrivalAirport`|Código do aeroporto de chegada. Mais informações em [IATA 3-Letter Codes](http://www.nationsonline.org/oneworld/IATA_Codes/airport_code_list.htm){:target="_blank"}|string|Não|3|
|`CustomConfiguration.Comments`|Comentários que a loja poderá associar a análise de fraude|string|Não|255|
|`CustomConfiguration.ScoreThreshold`|Nível aceitável de risco para cada produto|int|-|-|
|`MerchantDefinedData[n].Key`|Chave do campo definido junto ao provedor de antifraude <br/> [Tabela 34 - MerchantDefinedData(Cybersource)](https://braspag.github.io/manual/antifraude#tabela-34-merchantdefineddata-(cybersource))|int|Não|-|
|`MerchantDefinedData[n].Value`|Valor do campo definido junto ao provedor de antifraude <br/> [Tabela 34 - MerchantDefinedData(Cybersource)](https://braspag.github.io/manual/antifraude#tabela-34-merchantdefineddata-(cybersource))|var|Não|-|

### Resposta

``` json
{
   "TransactionId": "1eae3d39-a723-e811-80c3-0003ff21d83f",
   "Status": "Accept",
   "ProviderAnalysisResult": {
       "ProviderTransactionId": "5206061832306553904009",
       "ProviderStatus": "ACCEPT",
       "ProviderCode": "100",
       "ProviderRequestTransactionId": "AhjzbwSTGjifFZXHYduJEAFReTUyEoftDpA9+Ehk0kv9Atj2YEBMmAL2",
       "AfsReply": {
           "reasonCode": "100",
           "afsResult": "99",
           "hostSeverity": "3",
           "consumerLocalTime": "11:36:23",
           "afsFactorCode": "D^F^Z",
           "addressInfoCode": "COR-BA",
           "hotlistInfoCode": "NEG-AFCB^NEG-BA^NEG-CC^NEG-EM^NEG-PEM^NEG-SA^REV-PPH^REV-SUSP",
           "internetInfoCode": "FREE-EM^RISK-EM",
           "suspiciousInfoCode": "RISK-TB",
           "velocityInfoCode": "VEL-NAME",
           "scoreModelUsed": "default_lac"
       },
       "DecisionReply": {
           "casePriority": "3",
           "activeProfileReply": {},
           "velocityInfoCode": "GVEL-R1^GVEL-R2^GVEL-R4^GVEL-R6^GVEL-R7^GVEL-R9"
       }
   },
   "Links": [
       {
           "Method": "GET",
           "Href": "http://localhost:1316/Analysis/v2/1eae3d39-a723-e811-80c3-0003ff21d83f",
           "Rel": "Self"
       }
   ]
}
```

**Parâmetros no cabeçalho (header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|
|`Status`|201 Created|

**Parâmetros no corpo (body)**

|Parâmetro|Descrição|Tipo|
|:-|:-|:-:|
|`TransactionId`|Id da transação no Antifraude Gateway Braspag|GUID|
|`Status`|Status da transação no Antifraude Gateway Braspag <br/> [Tabela 19 - Status](https://braspag.github.io/manual/antifraude#tabela-19-status)|enum|
|`ProviderAnalysisResult.ProviderTransactionId`|Id da transação na Cybersource|string|
|`ProviderAnalysisResult.ProviderStatus`|Status da transação na Cybersource <br/> [Tabela 20 - ProviderStatus](https://braspag.github.io/manual/antifraude#tabela-20-providerstatus)|enum|
|`ProviderAnalysisResult.ProviderCode`|Código de retorno da Cybersouce <br/> [Tabela 21 - ProviderAnalysisResult.ProviderCode](https://braspag.github.io/manual/antifraude#tabela-21-provideranalysisresult.providercode)|int|
|`ProviderAnalysisResult.ProviderRequestTransactionId`|Id do request da transação na Cybersource|string|
|`ProviderAnalysisResult.Missing`|Campos faltantes na requisição enviada a Cybersource|string|
|`ProviderAnalysisResult.Invalid`|Campos com valores inválidos enviado a Cybersource|string|
|`ProviderAnalysisResult.AfsReply.AddressInfoCode`|Códigos indicam incompatibilidades entre os endereços de cobrança e entrega do comprador <br/> Os códigos são concatenados usando o caracter ^ Ex.: MM-A^MM-Z <br/>[Tabela 22 - ProviderAnalysisResult.AfsReply.AddressInfoCode](https://braspag.github.io/manual/antifraude#tabela-22-provideranalysisresult.afsreply.addressinfocode)|string|
|`ProviderAnalysisResult.AfsReply.AfsFactorCode`|Códigos que afetaram a pontuação da análise <br/> Os códigos são concatenados usando o caracter ^. Ex.: F^P <br/>[Tabela 23 - ProviderAnalysisResult.AfsReply.AfsFactorCode](https://braspag.github.io/manual/antifraude#tabela-23-provideranalysisresult.afsreply.afsfactorcode)|string|
|`ProviderAnalysisResult.AfsReply.AfsResult`|Score total calculado para o pedido|int|
|`ProviderAnalysisResult.AfsReply.BinCountry`|Código do país do BIN do cartão usado na análise. Mais informações em [ISO 2-Digit Alpha Country Code](https://www.iso.org/obp/ui){:target="_blank"}|string|
|`ProviderAnalysisResult.AfsReply.CardAccountType`|Tipo do cartão do comprador <br/>[Tabela 24 - ProviderAnalysisResult.AfsReply.CardAccountType](https://braspag.github.io/manual/antifraude#tabela-24-provideranalysisresult.afsreply.cardaccounttype)|string|
|`ProviderAnalysisResult.AfsReply.CardIssuer`|Nome do banco ou entidade emissora do cartão|string|
|`ProviderAnalysisResult.AfsReply.CardScheme`|Bandeira do cartão|string|
|`ProviderAnalysisResult.AfsReply.ConsumerLocalTime`|Horário local do comprador, calculado a partir da data da solicitação e do endereço de cobrança|string|
|`ProviderAnalysisResult.AfsReply.HostSeverity`|Nível de risco do domínio de e-mail do comprador, de 0 a 5, onde 0 é risco indeterminado e 5 representa o risco mais alto|int|
|`ProviderAnalysisResult.AfsReply.HotListInfoCode`|Códigos que indicam que os dados do comprador estão associados em listas de positivas ou negativas <br/> Os códigos são concatenados usando o caracter ^. Ex.: NEG-AFCB^NEG-CC <br/>[Tabela 25 - ProviderAnalysisResult.AfsReply.HotListInfoCode](https://braspag.github.io/manual/antifraude#tabela-25-provideranalysisresult.afsreply.hotlistinfocode)|string|
|`ProviderAnalysisResult.AfsReply.IdentityInfoCode`|Códigos que indicam mudanças de identidade excessivas <br/> Os códigos são concatenados usando o caracter ^. Ex.: COR-BA^MM-BIN <br/> [Tabela 26 - ProviderAnalysisResult.AfsReply.IdentityInfoCode](https://braspag.github.io//manual/antifraude#tabela-26-provideranalysisresult.afsreply.identityinfocode)|string|
|`ProviderAnalysisResult.AfsReply.InternetInfoCode`|Códigos que indicam problemas com o endereço de e-mail, o endereço IP ou o endereço de cobrança <br/> Os códigos são concatenados usando o caracter ^. Ex.: FREE-EM^RISK-EM <br/> [Tabela 27 - ProviderAnalysisResult.AfsReply.InternetInfoCode](https://braspag.github.io//manual/antifraude#tabela-27-provideranalysisresult.afsreply.internetinfocode)|string|
|`ProviderAnalysisResult.AfsReply.IpCity`|Nome da cidade do comprador obtido a partir do endereço de IP|string|
|`ProviderAnalysisResult.AfsReply.IpCountry`|Nome do país do comprador obtido a partir do endereço de IP|string|
|`ProviderAnalysisResult.AfsReply.IpRoutingMethod`|Método de roteamento do comprador obtido a partir do endereço de IP <br/> [Tabela 31 - ProviderAnalysisResult.AfsReply.IpRoutingMethod](https://braspag.github.io//manual/antifraude#tabela-31-provideranalysisresult.afsreply.iproutingmethod)|string|
|`ProviderAnalysisResult.AfsReply.IpState`|Nome do estado do comprador obtido a partir do endereço de IP|string|
|`ProviderAnalysisResult.AfsReply.PhoneInfoCode`|Códigos que indicam um problema com o número de telefone do comprador <br/> Os códigos são concatenados usando o caracter ^. Ex.: UNV-AC^RISK-AC <br/> [Tabela 28 - ProviderAnalysisResult.AfsReply.PhoneInfoCode](https://braspag.github.io//manual/antifraude#tabela-28-provideranalysisresult.afsreply.phoneinfocode)|string|
|`ProviderAnalysisResult.AfsReply.ReasonCode`|Código de retorno da Cybersouce <br/> [Tabela 21 - ProviderAnalysisResult.ProviderCode](https://braspag.github.io//manual/antifraude#tabela-21-provideranalysisresult.providercode)|int|
|`ProviderAnalysisResult.AfsReply.ScoreModelUsed`|Nome do modelo de score utilizado na análise. Caso não tenha nenhum modelo definido, o modelo padrão da Cybersource foi o utilizado|string|
|`ProviderAnalysisResult.AfsReply.SuspiciousInfoCode`|Códigos que indicam que o comprador forneceu potencialmente informações suspeitas <br/> Os códigos são concatenados usando o caracter ^. Ex.: RISK-TB^RISK-TS <br/> [Tabela 29 - ProviderAnalysisResult.AfsReply.SuspiciousInfoCode](https://braspag.github.io//manual/antifraude#tabela-29-provideranalysisresult.afsreply.suspiciousinfocode)|string|
|`ProviderAnalysisResult.AfsReply.VelocityInfoCode`|Códigos que indicam que o comprador tem uma alta frequência de compras <br/> Os códigos são concatenados usando o caracter ^. Ex.: VELV-SA^VELI-CC^VELSIP <br/> [Tabela 30 - ProviderAnalysisResult.AfsReply.VelocityInfoCode](https://braspag.github.io//manual/antifraude#tabela-30-provideranalysisresult.afsreply.velocityinfocode)|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.BrowserLanguage`|Linguagem do browser utilizado pelo comprador no momento da compra|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.ScreenResolution`|Resolução da tela do comprador no momento da compra|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.CookiesEnabled`|Flag identificando que o browser do comprador estava habilitado para armazenar cookies temporariamente no momento da compra|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.FlashEnabled`|Flag identificando que o browser do comprador habilitado a execução de conteúdos em Flash no momento da compra|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.Hash`|Hash gerado a partir dos dados coletados pelo script de fingerprint|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.ImagesEnabled`|Flag identificando que o browser do comprador estava com cache de imagens habilitado no momento da compra|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.JavascriptEnabled`|Flag identificando que o browser do comprador estava com a execução de sripts em Javascript habilitada no momento da compra|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.TrueIPAddress`|Flag identificando que o IP do comprador é real|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.TrueIPAddressCity`|Flag identificando que o IP do comprador é de fato da cidade que deveria ser mesmo|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.TrueIPAddressCountry`|Flag identificando que o IP do comprador é de fato do país que deveria ser mesmo|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.SmartId`|Identificador do dispositivo do comprador|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.SmartIdConfidenceLevel`|Identificador do dispositivo do comprador|string|
|`ProviderAnalysisResult.DecisionReply.ActiveProfileReply.DestinationQueue`|Quando modo verbose ativado, nome da fila para onde as transações não aceitas automaticamente são enviadas|string|
|`ProviderAnalysisResult.DecisionReply.ActiveProfileReply.Name`|Quando modo verbose ativado, nome do perfil selecionado na análise. Se não tiver nenhum, o perfil padrão foi selecionado|string|
|`ProviderAnalysisResult.DecisionReply.ActiveProfileReply.SelectedBy`|Quando modo verbose ativado, nome do seletor de regras que seleciona o perfil de regras|string|
|`ProviderAnalysisResult.DecisionReply.ActiveProfileReply.RulesTriggered[n].RuleId`|Quando modo verbose ativado, id da regra|enum|
|`ProviderAnalysisResult.DecisionReply.ActiveProfileReply.RulesTriggered[n].Decision`|Quando modo verbose ativado, decisão tomada pela regra <br/> [Tabela 32 - ProviderAnalysisResult.DecisionReply.ActiveProfileReply.RulesTriggered{n}.Decision](https://braspag.github.io//manual/antifraude#tabela-32-provideranalysisresult.decisionreply.activeprofilereply.rulestriggered[n].decision)|enum|
|`ProviderAnalysisResult.DecisionReply.ActiveProfileReply.RulesTriggered[n].Evaluation`|Quando modo verbose ativado, avaliação da regra <br/> [Tabela 33 - ProviderAnalysisResult.DecisionReply.ActiveProfileReply.RulesTriggered{n}.Evaluation](https://braspag.github.io//manual/antifraude#tabela-33-provideranalysisresult.decisionreply.activeprofilereply.rulestriggered[n].evaluation)|enum|
|`ProviderAnalysisResult.DecisionReply.ActiveProfileReply.RulesTriggered[n].Name`|Quando modo verbose ativado, nome da regra|string|
|`ProviderAnalysisResult.DecisionReply.CasePriority`|Define o nível de prioridade das regras ou perfis do lojista. O nível de prioridade varia de 1 (maior) a 5 (menor) e o valor padrão é 3, e este será atribuído caso não tenha definido a prioridade das regras ou perfis. Este campo somente será retornado se a loja for assinante do Enhanced Case Management|int|
|`ProviderAnalysisResult.DecisionReply.VelocityInfoCode`|Códigos de informação disparados pela análise. Estes códigos foram gerados no momento da criação das regras|string|

## Consultando uma transação Cybersource

Para saber o status de uma transação na Cybersource, recomendamos que você configure o [Post de Notificação](https://braspag.github.io//manual/antifraude#post-de-notifica%C3%A7%C3%A3o) e também faça a consulta por sondagem, que apresentamos a seguir:

### Requisição

<aside class="request"><span class="method get">GET</span> <span class="endpoint">analysis/v2/{Id}</span></aside>

**Parâmetros no cabeçalho (header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|
|`Authorization`|Bearer {access_token}|
|`MerchantId`|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|
|`RequestId`|nnnnnnnn-nnnn-nnnn-nnnn-nnnnnnnnnnnn|

### Resposta

``` json
{
   "TransactionId": "1eae3d39-a723-e811-80c3-0003ff21d83f",
   "Status": "Accept",
   "ProviderAnalysisResult": {
       "ProviderTransactionId": "5206061832306553904009",
       "ProviderStatus": "ACCEPT",
       "ProviderCode": "100",
       "ProviderRequestTransactionId": "AhjzbwSTGjifFZXHYduJEAFReTUyEoftDpA9+Ehk0kv9Atj2YEBMmAL2",
       "AfsReply": {
           "reasonCode": "100",
           "afsResult": "99",
           "hostSeverity": "3",
           "consumerLocalTime": "11:36:23",
           "afsFactorCode": "D^F^Z",
           "addressInfoCode": "COR-BA",
           "hotlistInfoCode": "NEG-AFCB^NEG-BA^NEG-CC^NEG-EM^NEG-PEM^NEG-SA^REV-PPH^REV-SUSP",
           "internetInfoCode": "FREE-EM^RISK-EM",
           "suspiciousInfoCode": "RISK-TB",
           "velocityInfoCode": "VEL-NAME",
           "scoreModelUsed": "default_lac"
       },
       "DecisionReply": {
           "casePriority": "3",
           "activeProfileReply": {},
           "velocityInfoCode": "GVEL-R1^GVEL-R2^GVEL-R4^GVEL-R6^GVEL-R7^GVEL-R9"
       }
   },
   "Links": [
       {
           "Method": "GET",
           "Href": "http://localhost:1316/Analysis/v2/1eae3d39-a723-e811-80c3-0003ff21d83f",
           "Rel": "Self"
       }
   ],
  "MerchantOrderId": "4493d42c-8732-4b13-aadc-b07e89732c26",
  "TotalOrderAmount": 15000,
  "TransactionAmount": 14000,
  "Currency": "BRL",
  "Provider": "Cybersource",
  "BraspagTransactionId":"a3e08eb2-2144-4e41-85d4-61f1befc7a3b",
  "Card": {
    "Number" : "4444555566667777",
    "Holder": "Holder Name",
    "ExpirationDate": "12/2023",
    "Cvv": "999",
    "Brand": "VISA"
  },
  "Billing": {
    "Street": "Rua Saturno",
    "Number": "12345",
    "Complement": "Sala 123",
    "Neighborhood": "Centro",
    "City": "Rio de Janeiro",
    "State": "RJ",
    "Country": "BR",
    "ZipCode": "20080123"
  },
  "Shipping": {
    "Street": "Rua Neturno",
    "Number": "30000",
    "Complement": "sl 123",
    "Neighborhood": "Centro",
    "City": "Rio de Janeiro",
    "State": "RJ",
    "Country": "BR",
    "ZipCode": "123456789",
    "FirstName": "João",
    "LastName": "Silva",
    "ShippingMethod": "SameDay",
    "Phone": "552121114700"
  },
  "Customer": {
    "MerchantCustomerId": "10050665740",
    "FirstName": "João",
    "LastName": "Silva",
    "BirthDate": "2016-12-09",
    "Email": "emailcomprador@dominio.com.br",
    "Phone": "552121114700",
    "Ip": "127.0.0.1",
    "BrowserHostName":"www.dominiobrowsercomprador.com.br",
    "BrowserCookiesAccepted":true,
    "BrowserEmail":"emailbrowsercomprador@dominio.com.br",
    "BrowserType":"Chrome 58 on Windows 10"
  },
  "CartItems": [
    {
      "ProductName": "Mouse",
      "Category": "EletronicGood",
      "UnitPrice": "12000",
      "Sku": "abc123",
      "Quantity": 1,
      "Risk":"Low",
      "AddressRiskVerify":"No",
      "HostHedge":"Low",
      "NonSensicalHedge":"Normal",
      "ObscenitiesHedge":"High",
      "TimeHedge":"Low",
      "PhoneHedge":"Normal",
      "VelocityHedge":"High"
    },
    {
      "ProductName": "Teclado",
      "Category": "EletronicGood",
      "UnitPrice": "96385",
      "MerchantItemId": "3",
      "Sku": "abc456",
      "Quantity": 1,
      "Risk": "High"
    }
  ],
  "MerchantDefinedData": [
    {
      "Key": "1",
      "Value": "Valor definido com o Provedor a ser enviado neste campo."
    },
    {
      "Key": "2",
      "Value": "Valor definido com o Provedor a ser enviado neste campo."
    },
    {
      "Key": "3",
      "Value": "Valor definido com o Provedor a ser enviado neste campo."
    }
  ],
  "Bank":{
    "Address": "Rua Marte, 29",
    "Code": "237",
    "Agency": "987654",
    "City": "Rio de Janeiro",
    "Country": "BR",
    "Name": "Bradesco",
    "SwiftCode": "789"
  },
  "FundTransfer":{
    "AccountNumber":"159753",
    "AccountName":"Conta particular",
    "BankCheckDigit":"51",
    "Iban":"123456789"
  },
  "Invoice":{
    "IsGift": false,
    "ReturnsAccept": true,
    "Tender": "Consumer"
  },
  "Airline": {
    "JourneyType": "OneWayTrip",
    "DepartureDateTime": "2018-01-09 18:00",
    "Passengers": [
    {
        "FirstName": "Fulano",
        "LastName": "Tal",
        "PassangerId": "",
        "PassengerType": "Adult",
        "Email": "email@mail.com",
        "Phone": "1234567890",
        "Status": "Gold",
        "Legs" : [
        {
            "ArrivalAirport": "AMS",
            "DepartureAirport": "GIG"
        }]
    }]
  }
}
```

**Parâmetros no cabeçalho (header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|
|`Status`|200 OK|

**Parâmetros no corpo (body)**

|Parâmetro|Descrição|Tipo|
|:-|:-|:-:|
|`TransactionId`|Id da transação no Antifraude Gateway Braspag|GUID|
|`Status`|Status da transação no Antifraude Gateway Braspag <br/> [Tabela 19 - Status](https://braspag.github.io//manual/antifraude#tabela-19-status)|enum|
|`ProviderAnalysisResult.ProviderTransactionId`|Id da transação na Cybersource|string|
|`ProviderAnalysisResult.ProviderStatus`|Status da transação na Cybersource <br/> [Tabela 20 - ProviderStatus](https://braspag.github.io//manual/antifraude#tabela-20-providerstatus)|enum|
|`ProviderAnalysisResult.ProviderCode`|Código de retorno da Cybersouce <br/> [Tabela 21 - ProviderAnalysisResult.ProviderCode](https://braspag.github.io//manual/antifraude#tabela-21-provideranalysisresult.providercode)|int|
|`ProviderAnalysisResult.ProviderRequestTransactionId`|Id do request da transação na Cybersource|string|
|`ProviderAnalysisResult.Missing`|Campos faltantes na requisição enviada a Cybersource|string|
|`ProviderAnalysisResult.Invalid`|Campos com valores inválidos enviado a Cybersource|string|
|`ProviderAnalysisResult.AfsReply.AddressInfoCode`|Códigos indicam incompatibilidades entre os endereços de cobrança e entrega do comprador <br/> Os códigos são concatenados usando o caracter ^ Ex.: MM-A^MM-Z <br/>[Tabela 22 - ProviderAnalysisResult.AfsReply.AddressInfoCode](https://braspag.github.io//manual/antifraude#tabela-22-provideranalysisresult.afsreply.addressinfocode)|string|
|`ProviderAnalysisResult.AfsReply.AfsFactorCode`|Códigos que afetaram a pontuação da análise <br/> Os códigos são concatenados usando o caracter ^. Ex.: F^P <br/>[Tabela 23 - ProviderAnalysisResult.AfsReply.AfsFactorCode](https://braspag.github.io//manual/antifraude#tabela-23-provideranalysisresult.afsreply.afsfactorcode)|string|
|`ProviderAnalysisResult.AfsReply.AfsResult`|Score total calculado para o pedido|int|
|`ProviderAnalysisResult.AfsReply.BinCountry`|Código do país do BIN do cartão usado na análise. Mais informações em [ISO 2-Digit Alpha Country Code](https://www.iso.org/obp/ui)|string|
|`ProviderAnalysisResult.AfsReply.CardAccountType`|Tipo do cartão do comprador <br/>[Tabela 24 - ProviderAnalysisResult.AfsReply.CardAccountType](https://braspag.github.io//manual/antifraude#tabela-24-provideranalysisresult.afsreply.cardaccounttype)|string|
|`ProviderAnalysisResult.AfsReply.CardIssuer`|Nome do banco ou entidade emissora do cartão|string|
|`ProviderAnalysisResult.AfsReply.CardScheme`|Bandeira do cartão|string|
|`ProviderAnalysisResult.AfsReply.ConsumerLocalTime`|Horário local do comprador, calculado a partir da data da solicitação e do endereço de cobrança|string|
|`ProviderAnalysisResult.AfsReply.HostSeverity`|Nível de risco do domínio de e-mail do comprador, de 0 a 5, onde 0 é risco indeterminado e 5 representa o risco mais alto|int|
|`ProviderAnalysisResult.AfsReply.HotListInfoCode`|Códigos que indicam que os dados do comprador estão associados em listas de positivas ou negativas <br/> Os códigos são concatenados usando o caracter ^. Ex.: NEG-AFCB^NEG-CC <br/>[Tabela 25 - ProviderAnalysisResult.AfsReply.HotListInfoCode](https://braspag.github.io//manual/antifraude#tabela-25-provideranalysisresult.afsreply.hotlistinfocode)|string|
|`ProviderAnalysisResult.AfsReply.IdentityInfoCode`|Códigos que indicam mudanças de identidade excessivas <br/> Os códigos são concatenados usando o caracter ^. Ex.: COR-BA^MM-BIN <br/> [Tabela 26 - ProviderAnalysisResult.AfsReply.IdentityInfoCode](https://braspag.github.io//manual/antifraude#tabela-26-provideranalysisresult.afsreply.identityinfocode)|string|
|`ProviderAnalysisResult.AfsReply.InternetInfoCode`|Códigos que indicam problemas com o endereço de e-mail, o endereço IP ou o endereço de cobrança <br/> Os códigos são concatenados usando o caracter ^. Ex.: COR-BA^MM-BIN <br/> [Tabela 27 - ProviderAnalysisResult.AfsReply.InternetInfoCode](https://braspag.github.io//manual/antifraude#tabela-27-provideranalysisresult.afsreply.internetinfocode)|string|
|`ProviderAnalysisResult.AfsReply.IpCity`|Nome da cidade do comprador obtido a partir do endereço de IP|string|
|`ProviderAnalysisResult.AfsReply.IpCountry`|Nome do país do comprador obtido a partir do endereço de IP|string|
|`ProviderAnalysisResult.AfsReply.IpRoutingMethod`|Método de roteamento do comprador obtido a partir do endereço de IP <br/> [Tabela 31 - ProviderAnalysisResult.AfsReply.IpRoutingMethod](https://braspag.github.io//manual/antifraude#tabela-31-provideranalysisresult.afsreply.iproutingmethod)|string|
|`ProviderAnalysisResult.AfsReply.IpState`|Nome do estado do comprador obtido a partir do endereço de IP|string|
|`ProviderAnalysisResult.AfsReply.PhoneInfoCode`|Códigos que indicam um problema com o número de telefone do comprador <br/> Os códigos são concatenados usando o caracter ^. Ex.: UNV-AC^RISK-AC <br/> [Tabela 28 - ProviderAnalysisResult.AfsReply.PhoneInfoCode](https://braspag.github.io//manual/antifraude#tabela-28-provideranalysisresult.afsreply.phoneinfocode)|string|
|`ProviderAnalysisResult.AfsReply.ReasonCode`|Código de retorno da Cybersouce <br/> [Tabela 21 - ProviderAnalysisResult.ProviderCode](https://braspag.github.io//manual/antifraude#tabela-21-provideranalysisresult.providercode)|int|
|`ProviderAnalysisResult.AfsReply.ScoreModelUsed`|Nome do modelo de score utilizado na análise. Caso não tenha nenhum modelo definido, o modelo padrão da Cybersource foi o utilizado|string|
|`ProviderAnalysisResult.AfsReply.SuspiciousInfoCode`|Códigos que indicam que o comprador forneceu potencialmente informações suspeitas <br/> Os códigos são concatenados usando o caracter ^. Ex.: RISK-TB^RISK-TS <br/> [Tabela 29 - ProviderAnalysisResult.AfsReply.SuspiciousInfoCode](https://braspag.github.io//manual/antifraude#tabela-29-provideranalysisresult.afsreply.suspiciousinfocode)|string|
|`ProviderAnalysisResult.AfsReply.VelocityInfoCode`|Códigos que indicam que o comprador tem uma alta frequência de compras <br/> Os códigos são concatenados usando o caracter ^. Ex.: VELV-SA^VELI-CC^VELSIP <br/> [Tabela 30 - ProviderAnalysisResult.AfsReply.VelocityInfoCode](https://braspag.github.io//manual/antifraude#tabela-30-provideranalysisresult.afsreply.velocityinfocode)|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.BrowserLanguage`|Linguagem do browser utilizado pelo comprador no momento da compra|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.ScreenResolution`|Resolução da tela do comprador no momento da compra|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.CookiesEnabled`|Flag identificando que o browser do comprador estava habilitado para armazenar cookies temporariamente no momento da compra|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.FlashEnabled`|Flag identificando que o browser do comprador habilitado a execução de conteúdos em Flash no momento da compra|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.Hash`|Hash gerado a partir dos dados coletados pelo script de fingerprint|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.ImagesEnabled`|Flag identificando que o browser do comprador estava com cache de imagens habilitado no momento da compra|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.JavascriptEnabled`|Flag identificando que o browser do comprador estava com a execução de sripts em Javascript habilitada no momento da compra|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.TrueIPAddress`|Flag identificando que o IP do comprador é real|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.TrueIPAddressCity`|Flag identificando que o IP do comprador é de fato da cidade que deveria ser mesmo|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.TrueIPAddressCountry`|Flag identificando que o IP do comprador é de fato do país que deveria ser mesmo|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.SmartId`|Identificador do dispositivo do comprador|string|
|`ProviderAnalysisResult.AfsReply.DeviceFingerprint.SmartIdConfidenceLevel`|Identificador do dispositivo do comprador|string|
|`ProviderAnalysisResult.DecisionReply.ActiveProfileReply.DestinationQueue`|Quando modo verbose ativado, nome da fila para onde as transações não aceitas automaticamente são enviadas|string|
|`ProviderAnalysisResult.DecisionReply.ActiveProfileReply.Name`|Quando modo verbose ativado, nome do perfil selecionado na análise. Se não tiver nenhum, o perfil padrão foi selecionado|string|
|`ProviderAnalysisResult.DecisionReply.ActiveProfileReply.SelectedBy`|Quando modo verbose ativado, nome do seletor de regras que seleciona o perfil de regras|string|
|`ProviderAnalysisResult.DecisionReply.ActiveProfileReply.RulesTriggered[n].RuleId`|Quando modo verbose ativado, id da regra|enum|
|`ProviderAnalysisResult.DecisionReply.ActiveProfileReply.RulesTriggered[n].Decision`|Quando modo verbose ativado, decisão tomada pela regra <br/> [Tabela 30 - ProviderAnalysisResult.DecisionReply.ActiveProfileReply.RulesTriggered{n}.Decision](https://braspag.github.io//manual/antifraude#tabela-32-provideranalysisresult.decisionreply.activeprofilereply.rulestriggered[n].decision)|enum|
|`ProviderAnalysisResult.DecisionReply.ActiveProfileReply.RulesTriggered[n].Evaluation`|Quando modo verbose ativado, avaliação da regra <br/> [Tabela 33 - ProviderAnalysisResult.DecisionReply.ActiveProfileReply.RulesTriggered{n}.Evaluation](https://braspag.github.io//manual/antifraude#tabela-33-provideranalysisresult.decisionreply.activeprofilereply.rulestriggered[n].evaluation)|enum|
|`ProviderAnalysisResult.DecisionReply.ActiveProfileReply.RulesTriggered[n].Name`|Quando modo verbose ativado, nome da regra|string|
|`ProviderAnalysisResult.DecisionReply.CasePriority`|Define o nível de prioridade das regras ou perfis do lojista. O nível de prioridade varia de 1 (maior) a 5 (menor) e o valor padrão é 3, e este será atribuído caso não tenha definido a prioridade das regras ou perfis. Este campo somente será retornado se a loja for assinante do Enhanced Case Management|string|
|`ProviderAnalysisResult.DecisionReply.VelocityInfoCode`|Códigos de informação disparados pela análise. Estes códigos foram gerados no momento da criação das regras|string|
|`MerchantOrderId` |Número do pedido da loja|string|
|`TotalOrderAmount`|Valor total do pedido em centavos <br/> Ex: 123456 = r$ 1.234,56|long|
|`TransactionAmount`|Valor da transação financeira em centavos <br/> Ex: 150000 = r$ 1.500,00|long|
|`Currency`|Moeda. Maiores informações em [ISO 4217 Currency Codes](https://www.iso.org/iso-4217-currency-codes.html){:target="_blank"}|string|
|`Provider`|Provedor da solução de antifraude <br/> [Tabela 1 - Provider](https://braspag.github.io//manual/antifraude#tabela-1-provider)|enum|
|`BraspagTransactionId`|Id da transação no Pagador da Braspag|guid|
|`Tid`|Id da transação na adquirente <br/>|string|
|`Nsu`|Número sequencial único da transação na adquirente <br/>|string|
|`AuthorizationCode`|Código de autorização da transação na adquirente <br/>|string|
|`SaleDate`|Data da autorização da transação na adquirente <br/>|datetime|
|`Card.Number`|Número do cartão de crédito|string|
|`Card.Holder`|Nome do cartão de crédito|string|
|`Card.ExpirationDate`|Data de expiração do cartão de crédito <br/> Ex.: 01/2023|string|
|`Card.Brand`|Bandeira do cartão de crédito <br/> [Tabela 3 - Card.Brand](https://braspag.github.io//manual/antifraude#tabela-3-card.brand)|enum|
|`Card.Save`|Indica se os dados do cartão de crédito serão armazenados no Cartão Protegido|booleano|
|`Card.Token`|Identificador do cartão de crédito salvo no Cartão Protegido|GUID|
|`Card.Alias`|Alias (apelido) do cartão de crédito salvo no Cartão Protegido|string|
|`Billing.Street`|Logradouro do endereço de cobrança|string|
|`Billing.Number`|Número do endereço de cobrança|string|
|`Billing.Complement`|Complemento do endereço de cobrança|string|
|`Billing.Neighborhood`|Bairro do endereço de cobrança|string|
|`Billing.City`|Cidade do endereço de cobrança|string|
|`Billing.State`|Estado do endereço de cobrança|string|
|`Billing.Country`|País do endereço de cobrança. Mais informações em [ISO 2-Digit Alpha Country Code](https://www.iso.org/obp/ui){:target="_blank"}|string|
|`Billing.ZipCode`|Código postal do endereço de cobrança|string|
|`Shipping.Street`|Logradouro do endereço de entrega|string|
|`Shipping.Number`|Número do endereço de entrega|string|
|`Shipping.Complement`|Complemento do endereço de entrega|string|
|`Shipping.Neighborhood`|Bairro do endereço de entrega|string|
|`Shipping.City`|Cidade do endereço de entrega|string|
|`Shipping.State`|Estado do endereço de entrega|string|
|`Shipping.Country`|País do endereço de entrega. Mais informações em [ISO 2-Digit Alpha Country Code](https://www.iso.org/obp/ui){:target="_blank"}|string|
|`Shipping.ZipCode`|Código postal do endereço de entrega|string|
|`Shipping.FirstName`|Primeiro nome do responsável a receber o produto no endereço de entrega|string|
|`Shipping.LastName`|Último do nome do responsável a receber o produto no endereço de entrega|string|
|`Shipping.Phone`|Número do telefone do responsável a receber o produto no endereço de entrega <br/> Ex.: 552121114700|string|
|`Shipping.ShippingMethod`|Meio de entrega do pedido <br/> [Tabela 4 - ShippingMethod](https://braspag.github.io//manual/antifraude#tabela-4-shippingmethod)|enum|
|`Customer.MerchantCustomerId`|Número do documento de identificação do comprador, CPF ou CNPJ|string|
|`Customer.FirstName`|Primeiro nome do comprador|string|
|`Customer.LastName`|Último nome do comprador|string|
|`Customer.BirthDate`|Data de nascimento do comprador <br/> Ex.: 1983-10-01|data|
|`Customer.Email`|E-mail do comprador|string|
|`Customer.Ip`|Endereço de IP do comprador|string|
|`Customer.Phone`|Número do telefone do comprador <br/> Ex.: 552121114700|string|
|`Customer.BrowserHostName`|Nome do host informado pelo browser do comprador e identificado através do cabeçalho HTTP|string|
|`Customer.BrowserCookiesAccepted`|Identifica se o browser do comprador aceita cookies ou não|booleano|
|`Customer.BrowserEmail`|E-mail registrado no browser do comprador. Pode diferenciar do e-mail cadastrado (`Customer.Email`)|string|
|`Customer.BrowserType`|Nome do browser utilizado pelo comprador e identificado através do cabeçalho HTTP|string|
|`CartItem[n].ProductName`|Nome do produto|string|
|`CartItem[n].Category`|Categoria do produto <br/> [Tabela 36 - CartItem{n}.Category](https://braspag.github.io//manual/antifraude#tabela-36-cartitem[n].category)|enum|
|`CartItem[n].Risk`|Nível de risco do produto associado a quantidade de chargebacks <br/> [Tabela 10 - CartItem{n}.Risk](https://braspag.github.io//manual/antifraude#tabela-10-cartitem[n].risk)|enum|
|`CartItem[n].UnitPrice`|Preço unitário do produto <br/> Ex: 10950 = r$ 109,50|long|
|`CartItem[n].Sku`|Sku do produto|string|
|`CartItem[n].Quantity`|Quantidade do produto|int|
|`CartItem[n].AddressRiskVerify`|Identifica que avaliará os endereços de cobrança e entrega para diferentes cidades, estados ou países <br/> [Tabela 11 - CartItem{n}.AddressRiskVerify](https://braspag.github.io//manual/antifraude#tabela-11-cartitem[n].addressriskverify)|enum|
|`CartItem[n].HostHedge`|Nível de importância dos endereços de IP e e-mail do comprador na análise de fraude <br/> [Tabela 12 - CartItem{n}.HostHedge](https://braspag.github.io//manual/antifraude#tabela-12-cartitem[n].hosthedge)|enum|
|`CartItem[n].NonSensicalHedge`|Nível de importância das verificações sobre os dados do comprador sem sentido na análise de fraude <br/> [Tabela 13 - CartItem{n}.NonSensicalHedge](https://braspag.github.io//manual/antifraude#tabela-13-cartitem[n].nonsensicalhedge)|enum|
|`CartItem[n].ObscenitiesHedge`|Nível de importância das verificações sobre os dados do comprador com obscenidade na análise de fraude <br/> [Tabela 14 - CartItem{n}.ObscenitiesHedge](https://braspag.github.io//manual/antifraude#tabela-14-cartitem[n].obscenitieshedge)|enum|
|`CartItem[n].TimeHedge`|Nível de importância da hora do dia na análise de fraude que o comprador realizou o pedido <br/> [Tabela 15 - CartItem{n}.TimeHedge](https://braspag.github.io//manual/antifraude#tabela-15-cartitem[n].timehedge)|enum|
|`CartItem[n].PhoneHedge`|Nível de importância das verificações sobre os números de telefones do comprador na análise de fraude <br/> [Tabela 16 - CartItem{n}.PhoneHedge](https://braspag.github.io//manual/antifraude#tabela-16-cartitem[n].phonehedge)|enum|
|`CartItem[n].VelocityHedge`|Nível de importância da frequência de compra do comprador na análise de fraude dentros dos 15 minutos anteriores <br/> [Tabela 17 - CartItem{n}.VelocityHedge](https://braspag.github.io//manual/antifraude#tabela-17-cartitem[n].velocityhedge)|enum|
|`Bank.Name`|Nome do banco do comprador|string|
|`Bank.Code`|Código do banco do comprador|string|
|`Bank.Agency`|Agência do banco do comprador|string|
|`Bank.Address`|Endereço do banco do comprador|string|
|`Bank.City`|Cidade onde está localizado o banco do comprador|string|
|`Bank.Country`|País onde está localizado o banco do comprador <br/> Mais informações em [ISO 2-Digit Alpha Country Code](https://www.iso.org/obp/ui){:target="_blank"}|string|
|`Bank.SwiftCode`|Código identificador único do banco do comprador|string|
|`FundTransfer.AccountName`|Nome vinculado a conta bancária|string|
|`FundTransfer.AccountNumber`|Número da conta bancária do comprador|string|
|`FundTransfer.BankCheckDigit`|Código utilizado para validar a conta bancária do comprador|string|
|`FundTransfer.Iban`|Número internacional da conta bancária do comprador (IBAN)|string|
|`Invoice.IsGift`|Indica se o pedido realizado pelo comprador é para presente|booleano|
|`Invoice.ReturnsAccepted`|Indica se o pedido realizado pelo comprador pode ser devolvido a loja|booleano|
|`Invoice.Tender`|Forma de pagamento utilizada pelo comprador <br/> [Tabela 18 - Invoice.Tender](https://braspag.github.io//manual/antifraude#tabela-18-invoice.tender)|enum|
|`Airline.JourneyType`|Tipo de viagem <br/> [Tabela 7 - Airline.JourneyType](https://braspag.github.io//manual/antifraude#tabela-7-airline.journeytype)|enun|
|`Airline.DepartureDateTime`|Data e hora de partida <br/> Ex.: 2018-03-31 19:16:38|datetime|
|`Airline.Passengers[n].FirstName`|Primeiro nome do passageiro|string|
|`Airline.Passengers[n].LastName`|Último nome do passageiro|string|
|`Airline.Passengers[n].PassengerId`|Identificador do passageiro a quem a passagem foi emitida|string|
|`Airline.Passengers[n].PassengerType`|Tipo do passageiro <br/> [Tabela 8 - Airline.Passengers{n}.PassengerType](https://braspag.github.io//manual/antifraude#tabela-8-airline.passengers[n].passengertype)|enum|
|`Airline.Passengers[n].Phone`|Telefone do passageiro <br/> Ex.: 552121114700|string|
|`Airline.Passengers[n].Email`|E-mail do passageiro|string|
|`Airline.Passengers[n].Status`|Classificação da empresa aérea <br/> [Tabela 9 - Airline.Passengers{n}.Status](https://braspag.github.io//manual/antifraude#tabela-9-airline.passengers[n].status)|enum|
|`Airline.Passengers[n].Legs[n].DepartureAirport`|Código do aeroporto de partida. Mais informações em [IATA 3-Letter Codes](http://www.nationsonline.org/oneworld/IATA_Codes/airport_code_list.htm){:target="_blank"}|string|
|`Airline.Passengers[n].Legs[n].ArrivalAirport`|Código do aeroporto de chegada. Mais informações em [IATA 3-Letter Codes](http://www.nationsonline.org/oneworld/IATA_Codes/airport_code_list.htm){:target="_blank"}|string|
|`CustomConfiguration.Comments`|Comentários que a loja poderá associar a análise de fraude|string|
|`CustomConfiguration.ScoreThreshold`|Nível aceitável de risco para cada produto|int|
|`MerchantDefinedData[n].Key`|Chave do campo definido junto ao provedor de antifraude <br/> [Tabela 34 - MerchantDefinedData(Cybersource)](https://braspag.github.io//manual/antifraude#tabela-34-merchantdefineddata-(cybersource))|int|não|-|
|`MerchantDefinedData[n].Value`|Valor do campo definido junto ao provedor de antifraude <br/> [Tabela 34 - MerchantDefinedData(Cybersource)](https://braspag.github.io//manual/antifraude#tabela-34-merchantdefineddata-(cybersource))|var|não|-|

## Update de status Cybersource

Em caso de revisão manual pelo próprio e-commerce, recomendamos que o resultado da revisão (novo status) seja enviado para a Cybersource a fim de refinar o processo de análise de fraude da sua loja.

Siga o exemplo de requisição a seguir para alterar o status de transações:

* De *Review* (em revisão) para *Accept* (aceita) ou *Reject* (rejeitada);
* De *Accept* (aceita) para *Reject* (rejeitada).

### Requisição

Para solicitar o update de status você deve gerar o token de acesso (access_token) e enviá-lo no header da requisição.

<aside class="request"><span class="method patch">PATCH</span> <span class="endpoint">analysis/v2/{id}</span></aside>

``` json
{
    "Status": "Accept",
    "Comments": "Dados do cliente OK"
}
```

**Parâmetros no cabeçalho (header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|
|`Authorization`|Bearer {access_token}|
|`MerchantId`|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|
|`RequestId`|nnnnnnnn-nnnn-nnnn-nnnn-nnnnnnnnnnnn|

**Parâmetros no corpo (body)**

|Parâmetro|Descrição|Tipo|Obrigatório|Tamanho|
|:-|:-|:-:|:-:|-:|
|`Status`|Novo status da transação. Accept ou Reject|string|Sim|-|
|`Comments`|Comentário associado a mudança de status|string|Não|255|

### Resposta

* Quando a transação for recebida para processamento

``` json
{
    "Status": "Accept",
    "ChangeStatusResponse": {
        "Status": "OK",
        "Message": "Change Status request successfully received. New status: Accept."
    }
}
```

**Parâmetros no cabeçalho (header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|
|`Status`|200 OK|

**Parâmetros no corpo (body)**

|Parâmetro|Descrição|
|:-|:-|
|`Status`|Novo status da transação|string|
|`ChangeStatusResponse.Status`|Identifica que a Cybersource recebeu a solicitação de alteração de status|string|
|`ChangeStatusResponse.Message`|Mensagem contendo conteúdo da operação realizada|string|

* Quando a transação não for encontrada na base de dados.
<br/><br/>
**Parâmetros no cabeçalho (header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|
|`Status`|404 Not Found|

* Quando a transação não estiver elegivel para alteração de status.
<br/><br/>
**Parâmetros no cabeçalho (header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|
|`Status`|400 Bad Request|

* Quando o novo status enviado for diferente de Accept ou Reject.
<br/><br/>
**Parâmetros no cabeçalho (header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|
|`Status`|400 Bad Request|

* Quando o tipo ou tamanho de algum campo não for enviado conforme especificado no manual.

``` json
{
    "Message": "The request is invalid.",
    "ModelState": {
        "request.Status": [
            "Error converting value \"Review\" to type 'Antifraude.Domain.Enums.StatusType'. Path 'Status', line 2, position 16."
        ],
        "request.Comments": [
            "The field Comments must be a string or array type with a maximum length of '255'."
        ]
    }
}
```

**Parâmetros no cabeçalho (header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|
|`Status`|400 Bad Request|

**Parâmetros no corpo (body)**

|Parâmetro|Descrição|
|:-|:-|
|`Message`|Mensagem informando que o request é inválido|
|`ModelState`|Coleção que conterá mensagens com os campos que não estejam de acordo com o tipo, domínio ou tamanho conforme especificado no manual|

## Fingerprint com a Cybersource

### O que é o Fingerprint?

O Fingerprint é a identificação digital do dispositivo do comprador. Essa identificação é composta por uma série de dados coletados na página de checkout do site ou aplicativo como:

* IP do dispositivo do comprador;
* Versão do navegador;
* Sistema operacional;
* Compatibilidade entre idioma e país.<br/>
<br/>

O Fingerprint faz a identificação do dispositivo usado por sessão de navegação e persiste por aproximadamente 24 horas. Se a página for fechada e o comprador retornar ao site abrindo uma nova página, ou se fechar o aplicativo e abrir novamente, você deverá gerar uma nova sessão e uma nova identificação de sessão.

O Fingerprint é importante para a análise de fraude porque, muitas vezes, somente os dados do carrinho não são suficientes para garantir uma análise assertiva. Os dados coletados pelo Fingerprint complementam a análise e aumentam a segurança da sua loja.

<aside class="notice">IMPORTANTE: Para atender aos requisitos da Lei Geral de Proteção de Dados (LGPD), inclua a informação sobre coleta de dados do dispositivo do comprador na política de cookies do seu e-commerce.</aside>

### Quem cria o Fingerprint?

Para análises via Cybersource, o Fingerprint é criado antes da requisição de análise de fraude pela **Threatmetrix**, empresa que faz a identificação do dispositivo.

Para estabelecer a comunicação entre a sua página de checkout e a Threatmetrix e enviar os dados do comprador, você precisa inserir um código do Fingerprint em seu e-commerce; leia mais sobre isso em [Como configurar o Fingerprint na Cybersource?](https://braspag.github.io//manual/antifraude#como-configurar-o-fingerprint-na-cybersource?)

### Fluxo do Fingerprint com a Cybersource

A criação do Fingerprint acontece separada da requisição de análise de fraude.

Veja a representação do fluxo de criação do Fingerprint e requisição de análise de fraude:

![Fluxo Fingerprint Cybersource]({{ site.baseurl_root }}/images/braspag/af/fluxo-fingerprint-cybersource.png)
 
**Etapa de criação do Fingerprint**

1. O comprador preenche os dados solicitados na página de checkout da loja (website ou aplicativo);
2. A página de checkout da loja, já configurada com o código do Fingerprint, coleta os dados do comprador e envia para a Threatmetrix solicitando a identificação do dispositivo (criação do Fingerprint);
3. A Threatmetrix cria o Fingerprint do dispositivo do comprador.

**Etapa de análise de fraude**

1. A loja envia a requisição de análise de fraude com o campo `Customer.BrowserFingerprint` para o Antifraude Gateway;
2. O Antifraude Gateway valida a requisição e solicita a análise de fraude para a Cybersource;
3. A Cybersource consulta o Fingerprint na Threatmetrix, realiza a análise de fraude e envia a recomendação (Accept/Reject/Review) para o Antifraude Gateway;
4. O Antifraude Gateway retorna o resultado da análise de fraude para a loja;
5. A loja retorna o status da transação (aprovada ou não aprovada) ao comprador.

### Onde enviar o Fingerprint?

Na requisição de análise de fraude com a Cybersource, o valor do parâmetro `Customer.BrowserFingerprint` será o `ProviderIdentifier` gerado pela loja.

Note que o valor do `Customer.BrowserFingerprint` não é o Fingerprint em si, mas sim uma indicação do Fingerprint da transação (`ProviderIdentifier`). Essa indicação será usada pela Cybersource para consultar o Fingerprint no serviço de identificação do dispositivo (Threatmetrix) e assim usá-lo para compor a análise de fraude.

> **Atenção**:
> <br>
> O campo de envio do identificador do Fingerprint é diferente quando a análise de fraude faz parte da requisição da transação, ou seja, para clientes integrados à API do Pagador, API E-commerce Cielo ou APIs do Split. Confira o campo de envio do `ProviderIdentifier` em cada situação:
> <br>
> * **Análise de fraude Cybersource usando a [API do Pagador](https://braspag.github.io//manual/braspag-pagador#pagamentos-com-an%C3%A1lise-de-fraude){:target="_blank"}**: envie o `ProviderIdentifier` no parâmetro `Payment.FraudAnalisys.FingerPrintId`;
> <br>
> * **Análise de fraude Cybersource usando a [API E-commerce Cielo](https://braspag.github.io//manual/api-ecommerce-cielo-af){:target="_blank"}**: envie o `ProviderIdentifier` no parâmetro `Payment.FraudAnalysis.Browser.BrowserFingerprint`;
> <br>
> * **Análise de fraude Cybersource para clientes [Split via API do Pagador](https://braspag.github.io//manual/split-de-pagamentos-pagador){:target="_blank"}**: envie o `ProviderIdentifier` no parâmetro `Payment.FraudAnalisys.FingerPrintId`;
> <br>
> * **Análise de fraude Cybersource para clientes [Split via API E-commerce Cielo](https://braspag.github.io//manual/split-de-pagamentos-cielo-e-commerce){:target="_blank"}**: envie o `ProviderIdentifier` no parâmetro `Payment.FraudAnalysis.Browser.BrowserFingerprint`.

### Como configurar o Fingerprint na Cybersource?

O Fingerprint consiste na implementação de um script na sua página de checkout (front-end), na parte onde o comprador preenche os dados cadastrais.

A configuração do Fingerprint será diferente para cada tipo de aplicação cliente ([web](https://braspag.github.io//manual/antifraude#configurando-o-fingerprint-na-cybersource-%E2%80%93-web), [Android](https://braspag.github.io//manual/antifraude#configurando-o-fingerprint-na-cybersource-%E2%80%93-android) ou [iOS](https://braspag.github.io//manual/antifraude#configurando-o-fingerprint-na-cybersource-%E2%80%93-ios)), mas as variáveis usadas são as mesmas; veja a seguir a tabela com as variáveis do Fingerprint.

#### Variáveis do Fingerprint

A tabela a seguir apresenta as variáveis para configuração do Fingerprint com a Threatmetrix e Cybersource.

|VARIÁVEL|DESCRIÇÃO|VALOR|FORMATO|TAMANHO|
|---|---|---|---|---|
|`org_id`| Indica o ambiente na Threatmetrix: Sandbox ou Produção.|Sandbox = 1snn5n9w<br>Produção = k8vif92e|String|08|
|`ProviderMerchantId`| Identificador da sua loja ou operação, fornecido pela Braspag, no formato braspag_nomedaloja.<br>**É diferente do MerchantId**. |Fornecido pela Braspag após a contratação.|String|30|
|`ProviderIdentifier`| Variável que você deve gerar para identificar a sessão. Recomendamos usar um GUID. É o valor que será enviado no campo `Customer.BrowserFingerprint`.|Personalizado|GUID ou String, na qual são aceitos inteiro, letra maiúscula ou minúscula, hífen e "\_" (*underscore*).|88|
|`session_id` (para web)| Concatenação das variáveis `ProviderMerchantId` e `ProviderIdentifier`. O valor do `session_id` irá compor a [URL da Threatmetrix](https://braspag.github.io//manual/antifraude#1.-preencha-a-url-da-threatmetrix) que será enviada no [script](https://braspag.github.io//manual/antifraude#2.-adicione-as-tags-ao-script) da integração web.| Personalizado| `ProviderMerchantIdProviderIdentifier` |118|
|`MyVariable` (para mobile)|Concatenação das variáveis `ProviderMerchantId` e `ProviderIdentifier`. Veja mais detalhes em [6. Crie a variável de identificação da sessão](https://braspag.github.io//manual/antifraude#6.-crie-a-vari%C3%A1vel-de-identifica%C3%A7%C3%A3o-da-sess%C3%A3o)| Personalizado | `ProviderMerchantIdProviderIdentifier`|118|

### Configurando o Fingerprint na Cybersource – Web

Você deverá inserir um script em JavaScript no código front-end da sua página de checkout.

#### 1. Preencha a URL da Threatmetrix

A URL da Threatmetrix será inserida no script e por isso deve ser corretamente preenchida.

![URL Threatmetrix]({{ site.baseurl_root }}/images/braspag/af/url-threatmetrix.png)

> O modelo da URL da Threatmetrix é **https://h.online-metrix.net/fp/tags.js?org_id=OrgId&session_id=ProviderMerchantIdProviderIdentifier**

Na URL, substitua os valores `OrgId`, `ProviderMerchantId` e `ProviderIdentifier` conforme a orientação da tabela de Variáveis da URL Threatmetrix.

#### 2. Adicione as tags ao script

Insira a URL preenchida no passo 1 nas tags `script` e `noscript` do modelo JavaScript.

O modelo do JavaScript está representado na imagem a seguir.

![Exemplo Código]({{ site.baseurl_root }}/images/braspag/af/exemplo-script-js.png)

> [Acesse o nosso GitHub](https://github.com/Braspag/braspag.github.io/blob/docs/_i18n/pt/_posts/antifraude/javascript-fingerprint-cybersource.js){:target="_blank"} para visualizar e copiar o modelo JavaScript.

* Insira a tag `script` dentro da tag `head` para uma performance correta:

&lt;`head`&gt;<br>
&lt;`script type="text/javascript" src="https://h.online-metrix.net/fp/tags.js?org_id=ProviderOrgId&session_id=ProviderMerchantIdProviderIdentifier"&gt;&lt;/script`&gt;<br>
&lt;`/head`&gt;

* Insira a tag `noscript` dentro da tag `body`, para que a coleta dos dados do dispositivo seja realizada mesmo se o JavaScript do browser estiver desabilitado. A tag `noscript` é uma redundância para colaborar com a coleta dos dados.

&lt;`body`&gt;<br>
&lt;`noscript`&gt;<br>
&lt;`iframe style="width: 100px; height: 100px; border: 0; psition:absolute; top: -5000px;" src="https://h.online-metrix.net/fp/tags?org_id=ProviderOrgId&session_id=ProviderMerchantIdProviderIdentifier"`&gt;&lt;`/iframe`&gt;<br>
&lt;`/noscript`&gt;<br>
&lt;`/body`&gt;

<aside class="warning">Certifique-se de copiar todos os dados corretamente e substitur as variáveis pelos respectivos valores.</aside>

#### 3. Aplique o modelo JavaScript

Insira o Javascript com as tags (passo 2) no código front-end da sua página de checkout.

Você deve colocar o código no checkout, na parte de preenchimento dos dados cadastrais. Assim, os dados que compõem o Fingerprint serão coletados enquanto o comprador preenche o formulário.

> Na requisição de análise de fraude, envie no campo `Customer.BrowserFingerprint` apenas o valor `ProviderIdentifier`.

### Configurando o Fingerprint na Cybersource – Android

#### 1. Adicione o SDK ao seu projeto

Faça o download do [SDK Android](https://github.com/Braspag/braspag.github.io/raw/bf88c72d069e15925b13227ce653df931f275d1d/files/braspag/antifraude/ThreatMetrix%20Android%20SDK%206.0-138_.zip){:target="_blank"}.

Em seguida, adicione o SDK ao seu projeto.

#### 2. Adicione as bibliotecas

Adicione as bibliotecas e dependências ao projeto:

* TMXProfiling-6.0-138.aar 
* MXProfilingConnections-6.0-138.aar
<br/>
<br/>
Saiba mais sobre a criação de bibliotecas no Android na documentação [Android for Developers](https://developer.android.com/studio/projects/android-library){:target="_blank"}.

#### 3. Inclua as permissões

No Manifest, você deverá incluir as seguintes permissões:

&lt;`uses-permission android:name="android.permission.INTERNET"`&gt;

&lt;`/uses-permission`&gt;

#### 4. Importe as bibliotecas

Importe as seguintes bibliotecas:

* import com.threatmetrix.TrustDefender.TMXConfig
* import com.threatmetrix.TrustDefender.TMXEndNotifier
* import com.threatmetrix.TrustDefender.TMXProfiling
* import com.threatmetrix.TrustDefender.TMXProfilingHandle
* import com.threatmetrix.TrustDefender.TMXProfilingOptions

#### 5. Parametrize o SDK

Você deverá parametrizar o SDK com os parâmetros a seguir:

`TMXConfig config = new TMXConfig()`

`.setOrgId("OrgId")`
<br/>
<br/>
No valor **“OrgId”**, indique o valor correspondente ao ambiente na Threatmetrix:

* Sandbox: “1snn5n9w”;
* Produção: “k8vif92e”.

`.setFPServer("h.online-metrix.net")`

`.setContext(getApplicationContext());`

`.setTimeout(20, TimeUnit.SECONDS)`

`TMXProfiling.getInstance().init(config);`

#### 6. Crie a variável de identificação da sessão

O valor `ProviderMerchantId` deve ser concatenado com a variável `ProviderIdentifier` (definida pelo seu e-commerce) para criar a identificação da sessão (`MyVariable`). 

`MyVariable` = `ProviderMerchantId` + `ProviderIdentifier`

**Exemplo:**

`MyVariable` = `braspag_XXXX` + `ProviderIdentifier`
<br/>
<br/>
> Na requisição de análise de fraude, envie no campo `Customer.BrowserFingerprint` apenas o valor `ProviderIdentifier`. Se o `ProviderIdentifier` gerado pelo seu e-commerce for “202201080949”, no campo `Customer.BrowserFingerprint` envie o valor "202201080949".

<aside class="notice">Recomendamos que a variável `ProviderIdentifier` seja um GUID.</aside>

#### 7. Implemente o Profiling

Implemente Profiling com EndNotifier.

`TMXProfilingOptions options = new TMXProfilingOptions().setCustomAttributes(null);options.setSessionID(MyVariable)`

`TMXProfilingHandle profilingHandle = TMXProfiling.getInstance().profile(options,new CompletionNotifier());`

`class CompletionNotifier implements TMXEndNotifier`

`{`
<br/>
`Override public void complete(TMXProfilingHandle.Result result)`
<br/>
`{ // Once Profile is done. Check the status code in the results dictionary, and use the session Id in the API.`
<br/>
`}`
<br/>
`}`

Faça o download do [material de apoio da Cybersource](https://github.com/Braspag/braspag.github.io/raw/bf88c72d069e15925b13227ce653df931f275d1d/files/braspag/antifraude/DecisionManagerDeviceFingerprint_v6.pdf){:target="_blank"}.

### Configurando o Fingerprint na Cybersource – iOS

#### 1. Adicione o SDK ao seu projeto

Faça o download do [SDK iOS](https://github.com/Braspag/braspag.github.io/raw/bf88c72d069e15925b13227ce653df931f275d1d/files/braspag/antifraude/ThreatMetrix%20iOS%20SDK%206.0-91_.zip){:target="_blank"}.

Em seguida, adicione o SDK ao seu projeto.

#### 2. Importe as bibliotecas e dependências

Adicione as seguintes bibliotecas e dependências ao seu projeto:

* import RLTMXProfiling
* import RLTMXProfilingConnections
* import RLTMXBehavioralBiometrics

#### 3. Parametrize o SDK

Parametrize o SDK com os parâmetros a seguir:

`self.profile.configure(configData:[`

`RLTMXOrgID : "OrgID",`
<br/>
<br/>
No valor “OrgID”, indique o valor correspondente ao ambiente na Threatmetrix:

* Sandbox: “1snn5n9w”;
* Produção: “k8vif92e”.

`RLTMXFingerprintServer : "h.online-metrix.net",`

`RLTMXProfileTimeout : self.profileTimeout,`

`RLTMXLocationServices : true,`

`RLTMXProfilingConnectionsInstance : profilingConnections,`

`])`

#### 4. Crie a variável de identificação da sessão

O valor `ProviderMerchantId` deve ser concatenado com a variável `ProviderIdentifier` (definida pelo e-commerce) para criar a identificação da sessão (`MyVariable`). 

`MyVariable` = `ProviderMerchantId` + `ProviderIdentifier`

**Exemplo:**

`MyVariable` = `braspag_XXXX` + `ProviderIdentifier`

`self.profile.sessionID` = @"`MyVariable`"
<br/>
<br/>
> Na requisição de análise de fraude, envie no campo `Customer.BrowserFingerprint` apenas o valor `ProviderIdentifier`. Se o `ProviderIdentifier` gerado pelo seu e-commerce for “202201080949”, no campo `Customer.BrowserFingerprint` envie o valor "202201080949".

<aside class="notice">Recomendamos que a variável `ProviderIdentifier` seja um GUID.</aside>

#### 5. Implemente o Profiling

Adicione a função doProfileRequest() à sua aplicação e especifique as seguintes opções:

`let profileHandle: RLTMXProfileHandle =`

`self.profile.profileDevice(profileOptions:[RLTMXCustomAttributes: [],`

`RLTMXSessionID: [MyVariable], callbackBlock:{(result: [AnyHashable : Any]?) -> Void in`

Faça o download do [material de apoio da Cybersource](https://github.com/Braspag/braspag.github.io/raw/bf88c72d069e15925b13227ce653df931f275d1d/files/braspag/antifraude/DecisionManagerDeviceFingerprint_v6.pdf){:target="_blank"}.

# Integração com a ACI Worldwide

## Analisando uma transação na ACI Worldwide

Quando o seu provedor de antifraude é a ACI Worldwide, envie o valor "RedShield" no campo `Provider`.

### Requisição

<aside class="request"><span class="method post">POST</span> <span class="endpoint">analysis/v2/</span></aside>

``` json
{
  "MerchantOrderId": "4493d42c-8732-4b13-aadc-b07e89732c26",
  "TotalOrderAmount": 15000,
  "TransactionAmount": 14000,
  "Currency": "BRL",
  "Provider": "RedShield",
  "OrderDate": "2016-12-09 12:35:58.852",
  "BraspagTransactionId":"a3e08eb2-2144-4e41-85d4-61f1befc7a3b",
  "Tid": "12345678910111216AB8",
  "Nsu": "951852",
  "AuthorizationCode":"T12345",
  "SaleDate": "2016-12-09 10:01:55.662",
  "SplitingPaymentMethod": "None",
  "IsRetryTransaction": false,
  "Card": {
    "Number" : "4444555566667777",
    "Holder": "Holder Name",
    "ExpirationDate": "12/2023",
    "Cvv": "999",
    "Brand": "VISA",
    "EciThreeDSecure": "5"
  },
  "Billing": {
    "Street": "Rua Neturno",
    "Number": "12345",
    "Complement": "Sala 123",
    "Neighborhood": "Centro",
    "City": "Rio de Janeiro",
    "State": "RJ",
    "Country": "BR",
    "ZipCode": "20080123"
  },
  "Shipping": {
    "Street": "Rua Saturno",
    "Number": "30000",
    "Complement": "sl 123",
    "Neighborhood": "Centro",
    "City": "Rio de Janeiro",
    "State": "RJ",
    "Country": "BR",
    "ZipCode": "123456789",
    "Email": "emailentrega@dominio.com.br",
    "FirstName": "João",
    "MiddleName": "P",
    "LastName": "Silvao",
    "ShippingMethod": "SameDay",
    "Phone": "552121114700",
    "WorkPhone": "552121114721",
    "Mobile": "5521998765432",
    "Comment": "Em frente ao 322"
  },
  "Customer": {
    "MerchantCustomerId": "10050665740",
    "FirstName": "João",
    "MiddleName": "P",
    "LastName": "Silva",
    "BirthDate": "1983-10-01",
    "Gender": "Male",
    "Email": "emailcomprador@dominio.com.br",
    "Phone": "552121114700",
    "WorkPhone": "552121114721",
    "Mobile": "5521998765432",
    "Ip": "127.0.0.1",
    "BrowserFingerprint": "04003hQUMXGB0poNf94lis1ztuLYRFk+zJ17aP79a9O8mWOBmEnKs6ziAo94ggAtBvKEN6/FI8Vv2QMAyHLnc295s0Nn8akZzRJtHwsEilYx1P+NzuNQnyK6+7x2OpjJZkl4NlfPt7h9d96X/miNlYT65UIY2PeH7sUAh9vKxMn1nlPu2MJCSi12NBBoiZbfxP1Whlz5wlRFwWJi0FRulruXQQGCQaJkXU7GWWZGI8Ypycnf7F299GIR12G/cdkIMFbm6Yf0/pTJUUz1vNp0X2Zw8QydKgnOIDKXq4HnEqNOos1c6njJgQh/4vXJiqy0MXMQOThNipDmXv9I185O+yC2f3lLEO0Tay66NZEyiLNePemJKSIdwO9O5ZtntuUkG6NTqARuHStXXfwp8cyGF4MPWLuvNvEfRkJupBy3Z8hSEMEK7ZWd2T2HOihQxRh4qp+NANqYKBTl3v6fQJAEKikeSQVeBN8sQqAL0BZFaIMzbrnMivi6m6JRQUIdvEt+MbJEPFc0LjRycC5ApUmJO+Aoo9VKL1B8ftMSQ1iq1uTKn16ZOmDpzZrZhMPbH83aV0rfB2GDXcjpghm9klVFOw7EoYzV7IDBIIRtgqG9KZ+8NH/z6D+YNUMLEUuK1N2ddqKbS5cKs2hplVRjwSv7x8lMXWE7VDaOZWB8+sD1cMLQtEUC0znzxZ4bpRaiSy4dJLxuJpQYAFUrDlfSKRv/eHV3QiboXLuw9Lm6xVBK8ZvpD5d5olGQdc+NgsqjFnAHZUE+OENgY4kVU9wB84+POrI4MkoD4iHJ5a1QF8AZkZDFo1m1h9Bl+J2Ohr6MkBZq8DG5iVaunHfxUdHou5GL7lS1H7r+8ctfDXi8AfOPjzqyODJQ74Aiel35TKTOWG8pq1WO6yzJ1GNmMuMWZBamlGXoG/imnjwHY9HQtQzpGfcm0cR8X2Fd1ngNFGLDGZlWOX0jWtOwU6XVGT37JFD9W/cx4kzI+mPNi65X5WFPYlDG9N0Lbh5nOj3u3DXqRCiKCUrsEkMt8z9fxO9pLLGVQUKIYR2wTw53CiWK96FOpPevDWtH2XR0QkfOd02D73n81x6hEMCy0s3hRLn08Th9FlNHDMJBqLj+Tz8rG2TtNki3mJC7Ass1MT2qnKBI77n6vsQkAp59TfbZm/tBXwAoYdLJXge8F/numhd5AvQ+6I8ZHGJfdN3qWndvJ2I7s5Aeuzb8t9//eNsm73fIa05XreFsNyfOq1vG2COftC6EEsoJWe5h5Nwu1x6PIKuCaWxLY+npfWgM0dwJPmSgPx7TNM31LyVNS65m83pQ+qMTRH6GRVfg7HAcS5fnS/cjdbgHxEkRmgkRq1Qs48sbX9QC8nOTD0ntb6FcJyEOEOVzmJtDqimkzDq+SXR1/63AYe4LEj+ogRgN+Z8HAFhGFzd/m6snVviELfRqJ4LLQIk9Y/fzqnsF6I5OGxfdT2sxxK2Vokpi3jWhCcEknw7dYlHYpOnCHZO7QVgjQTngF2mzKf4GeOF4ECFsWTgLy6HFEitfauYJt1Xh1NfZZerBMwXLFzdhzoTQxGlcXc8lZIoEG1BLYv/ScICf8Ft9PEtpEa+j0cDSlU99UoH2xknwR1W9MRGc5I/euE63/IMJTqguZ3YcnJpjSVnAGSpyz/0gKjypJ3L86rHFRGXt0QbmaXtSl2UmmjI0p0LCCdx7McatCFEVI6FwPpPV0ZSMv/jM75eBid1X/lTV4XNzjowzR/iFlKYMzHZtVO9hCBPKlTwblRXNn4MlvNm/XeSRQ+Mr0YV5w5CL5Z/tGyzqnaLPj/kOVdyfj8r2m5Bcrz4g/ieUIo8qRFv2T2mET46ydqaxi27G4ZYHj7hbiaIqTOxWaE07qMCkJw==",
    "Status": "NEW"
  },
  "CartItems": [
    {
      "ProductName": "Mouse",
      "UnitPrice": "6500",
      "MerchantItemId": "4",
      "Sku": "abc123",
      "Quantity": 1,
      "OriginalPrice": "7000",
      "GiftMessage": "Feliz aniversário!",
      "Description": "Uma description do Mouse",
      "ShippingInstructions": "Proximo ao 546",
      "ShippingMethod": "SameDay",
      "ShippingTrackingNumber": "123456"
    },
    {
      "ProductName": "Teclado",
      "UnitPrice": "7500",
      "MerchantItemId": "3",
      "Sku": "abc456",
      "Quantity": 1,
      "OriginalPrice": "8000",
      "GiftMessage": "Parabéns!",
      "Description": "Uma description do Teclado",
      "ShippingInstructions": "Proximo ao 123",
      "ShippingMethod": "SameDay",
      "ShippingTrackingNumber": "987654"
    }
  ],
  "CustomConfiguration": {
    "MerchantWebsite": "www.test.com"
  },
  "MerchantDefinedData": [
    {
      "Key": "USER_DATA4",
      "Value": "Valor definido com o Provedor a ser enviado neste campo."
    },
    {
      "Key": "Segment",
      "Value": "8999"
    },
    {
      "Key": "MerchantId",
      "Value": "Seller123456"
    }
  ],
  "Airline": {
    "ThirdPartyBooking": "Y",
    "Bookingtype": "Corporate",
    "TicketDeliveryMethod": "Delivery",
    "BookingReferenceNumber": "L5W4NW",
    "Passengers": [
    {
        "FirstName": "Fulano",
        "MiddleName": "D",
        "LastName": "Tal",
        "PassengerType": "Adult",
        "Email": "email@mail.com",
        "Phone": "1234567890",
        "TicketNumber": "123541",
        "LoyaltyMemberNumber": "159753852",
        "Legs" : [
        {
            "ArrivalAirport": "AMS",
            "DepartureAirport": "GIG",
            "ArrivalCountry": "NLD",
            "DepartureCountry": "BRA",
            "AirlineCode": "KLM",
            "DepartureDateTime": "2018-01-09 18:00",
            "ClassOfService": "Standard"
        }]
    }]
  }
}
```

**Parâmetros no cabeçalho (header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|
|`Authorization`|Bearer {access_token}|
|`MerchantId`|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|
|`RequestId`|nnnnnnnn-nnnn-nnnn-nnnn-nnnnnnnnnnnn|

**Parâmetros no corpo (body)**

|Parâmetro|Descrição|Tipo|Obrigatório|Tamanho|
|:-|:-|:-:|:-:|-:|
|`MerchantOrderId`|Número do pedido da loja|string|Sim|100|
|`TotalOrderAmount`|Valor total do pedido em centavos <br/> Ex: 123456 = r$ 1.234,56|long|Sim|-|
|`TransactionAmount`|Valor da transação financeira em centavos <br/> Ex: 150000 = r$ 1.500,00|long|Sim|-|
|`Currency`|Moeda. Maiores informações em [ISO 4217 Currency Codes](https://www.iso.org/iso-4217-currency-codes.html){:target="_blank"}|enum|-|-|
|`Provider`|Provedor da solução de antifraude <br/> [Tabela 1 - Provider]({{ site.baseurl_root }}manual/antifraude#tabela-1-provider)|enum|-|-|
|`OrderDate`|Data do pedido <br/> Ex.: 2016-12-09 19:16:38.155 <br/> Obs.: Caso não seja informada, uma data será gerada pela Braspag|datetime|Não|-|
|`BraspagTransactionId`|Id da transação no Pagador da Braspag|GUID|Não|-|
|`Tid`|Id da transação na adquirente <br/> Obs.: Caso você não possua integração com o Pagador Braspag, não terá como enviar o campo `BraspagTransactionId`, com isso é necessário o envio dos campos `Nsu`, `AuthorizationCode` e `SaleDate`, além deste em questão|string|Não|20|
|`Nsu`|Número sequencial único da transação na adquirente <br/> Obs.: Caso você não possua integração com o Pagador Braspag, não terá como enviar o campo `BraspagTransactionId`, com isso é necessário o envio dos campos `Tid`, `AuthorizationCode` e `SaleDate`, além deste em questão|string|Não|10|
|`AuthorizationCode`|Código de autorização da transação na adquirente <br/> Obs.: Caso você não possua integração com o Pagador Braspag, não terá como enviar o campo `BraspagTransactionId`, com isso é necessário o envio dos campos `Tid`, `Nsu` e `SaleDate`, além deste em questão|string|Não|10|
|`SaleDate`|Data da autorização da transação da transação na adquirente <br/> Obs.: Caso você não possua integração com o Pagador Braspag, não terá como enviar o campo `BraspagTransactionId`, com isso é necessário o envio dos campos `Tid`, `Nsu` e `AuthorizationCode`, além deste em questão|datetime|Não|-|
|`SplitingPaymentMethod`|Identifica se a autorização da transação é com um ou mais cartões ou com mais de um meio de pagamento <br/> [Tabela 2 - SplitingPaymentMethod]({{ site.baseurl_root }}manual/antifraude#tabela-2-splitingpaymentmethod)|enum|-|-|
|`IsRetryTransaction`|Retentativa de uma análise, e deverá ser enviado com valor igual a TRUE quando o código de retorno na primeira tentativa for igual a BP900|booleano|Não|-|
|`Card.Number`|Número do cartão de crédito|string|Sim|19|
|`Card.Holder`|Nome do cartão de crédito|string|Sim|50|
|`Card.ExpirationDate`|Data de expiração do cartão de crédito <br/> Ex.: 01/2023|string|Sim|7|
|`Card.Cvv`|Código de segurança do cartão de crédito|string|Sim|4|
|`Card.Brand`|Bandeira do cartão de crédito <br/> [Tabela 3 - Card.Brand]({{ site.baseurl_root }}manual/antifraude#tabela-3-card.brand)|enum|-|-|
|`Card.EciThreeDSecure`|Código do ECI (Eletronic Commerce Indicator) de autenticação|string|Não|1|
|`Card.Save`|Indica se os dados do cartão de crédito serão armazenados no Cartão Protegido|booleano|Não|-|
|`Card.Token`|Identificador do cartão de crédito salvo no Cartão Protegido|GUID|Não|-|
|`Card.Alias`|Alias (apelido) do cartão de crédito salvo no Cartão Protegido|string|Não|64|
|`Billing.Street`|Logradouro do endereço de cobrança|string|Não|24|
|`Billing.Number`|Número do endereço de cobrança|string|Não|5|
|`Billing.Complement`|Complemento do endereço de cobrança|string|Não|14|
|`Billing.Neighborhood`|Bairro do endereço de cobrança|string|Não|15|
|`Billing.City`|Cidade do endereço de cobrança|string|Não|20|
|`Billing.State`|Estado do endereço de cobrança|string|Não|2|
|`Billing.Country`|País do endereço de cobrança. Mais informações em [ISO 2-Digit Alpha Country Code](https://www.iso.org/obp/ui){:target="_blank"}|string|Não|2|
|`Billing.ZipCode`|Código postal do endereço de cobrança|string|Não|9|
|`Shipping.Street`|Logradouro do endereço de entrega|string|Não|24|
|`Shipping.Number`|Número do endereço de entrega|string|Não|5|
|`Shipping.Complement`|Complemento do endereço de entrega|string|Não|14|
|`Shipping.Neighborhood`|Bairro do endereço de entrega|string|Não|15|
|`Shipping.City`|Cidade do endereço de entrega|string|Não|20|
|`Shipping.State`|Estado do endereço de entrega|string|Não|2|
|`Shipping.Country`|País do endereço de entrega. Mais informações em [ISO 2-Digit Alpha Country Code](https://www.iso.org/obp/ui){:target="_blank"}|string|Não|2|
|`Shipping.ZipCode`|Código postal do endereço de entrega|string|Não|9|
|`Shipping.Email`|E-mail do responsável a receber o produto no endereço de entrega|string|Não|60|
|`Shipping.FirstName`|Primeiro nome do responsável a receber o produto no endereço de entrega|string|Não|30|
|`Shipping.MiddleName`|Primeira letra do nome do meio do responsável a receber o produto no endereço de entrega|string|Não|1|
|`Shipping.LastName`|Último do nome do responsável a receber o produto no endereço de entrega|string|Não|30|
|`Shipping.Phone`|Número do telefone do responsável a receber o produto no endereço de entrega <br/> Ex.: 552121114700|string|Não|19|
|`Shipping.WorkPhone`|Número do telefone de trabalho do responsável a receber o produto no endereço de entrega <br/> Ex.: 552121114701|string|Não|19|
|`Shipping.Mobile`|Número do celular do responsável a receber o produto no endereço de entrega <br/> Ex.: 5521987654321|string|Não|19|
|`Shipping.ShippingMethod`|Meio de entrega do pedido <br/> [Tabela 4 - ShippingMethod]({{ site.baseurl_root }}manual/antifraude#tabela-4-shippingmethod)|enum|-|-|
|`Shipping.Comment`|Referências do endereço de entrega|string|Não|160|
|`Customer.MerchantCustomerId`|Número do documento de identificação do comprador, CPF ou CNPJ|string|Sim|16|
|`Customer.FirstName`|Primeiro nome do comprador|string|Sim|30|
|`Customer.MiddleName`|Primeira letra do nome do comprador|string|Não|1|
|`Customer.LastName`|Último nome do comprador|string|Sim|30|
|`Customer.BirthDate`|Data de nascimento do comprador <br/> Ex.: 1983-10-01|data|Sim|-|
|`Customer.Gender`|Sexo do comprador <br/> [Tabela 5 - Customer.Gender](https://braspag.github.io/manual/antifraude#tabela-5-customer.gender)|string|Não|6|
|`Customer.Email`|E-mail do comprador|string|Não|60|
|`Customer.Ip`|Endereço de IP do comprador|string|Não|15|
|`Customer.Phone`|Número do telefone do comprador <br/> Ex.: 552121114700|string|Não|19|
|`Customer.WorkPhone`|Número do telefone do comprador <br/> Ex.: 552121114701|string|Não|19|
|`Customer.Mobile`|Número do celular do comprador <br/> Ex.: 5521987654321|string|Não|19|
|`Customer.Status`|Status do comprador na loja <br/> [Tabela 6 - Customer.Status](https://braspag.github.io/manual/antifraude#tabela-6-customer.status)|string|Não|8|
|`Customer.BrowserFingerPrint`|Impressão digital de dispositivos e geolocalização real do IP do comprador - [Configuração do Fingerprint](https://braspag.github.io/manual/antifraude#fingerprint-com-a-aci-worldwide)|string|Sim|6005|
|`CartItem[n].ProductName`|Nome do produto|string|Não|50|
|`CartItem[n].UnitPrice`|Preço unitário do produto <br/> Ex: 10950 = r$ 109,50|long|Não|-|
|`CartItem[n].OriginalPrice`|Preço original do produto <br/> Ex: 11490 = r$ 114,90|long|Não|-|
|`CartItem[n].MerchantItemId`|ID do produto na loja|string|Não|30|
|`CartItem[n].Sku`|Sku do produto|string|Não|12|
|`CartItem[n].Quantity`|Quantidade do produto|int|Não|-|
|`CartItem[n].GiftMessage`|Mensagem de presente|string|Não|160|
|`CartItem[n].Description`|Descrição do produto|string|Não|76|
|`CartItem[n].ShippingInstructions`|Instruções de entrega do produto|string|Não|160|
|`CartItem[n].ShippingMethod`|Meio de entrega do produto <br/> [Tabela 4 - ShippingMethod](https://braspag.github.io/manual/antifraude#tabela-4-shippingmethod)|enum|-|-|
|`CartItem[n].ShippingTranckingNumber`|Número de rastreamento do produto|string|Não|19|
|`Airline.ThirdPartyBooking`|Indica se a reserva foi agendada por terceiros, como por exemplo agências de turismo|booleano|Não|-|
|`Airline.BookingType`|Tipo de agendamento da reserva|string|Não|255|
|`Airline.TicketDeliveryMethod`|Tipo de entrega da passagem|string|Não|127|
|`Airline.BookingReferenceNumber`|Número de referêcia da reserva|string|Não|9|
|`Airline.Passengers[n].FirstName`|Primeiro nome do passageiro|string|Não|29|
|`Airline.Passengers[n].MiddleName`|Nome do meio do passageiro|string|Não|1|
|`Airline.Passengers[n].LastName`|Último nome do passageiro|string|Não|28|
|`Airline.Passengers[n].PassengerType`|Tipo do passageiro <br/> [Tabela 8 - Airline.Passengers{n}.PassengerType](https://braspag.github.io/manual/antifraude#tabela-8-airline.passengers[n].passengertype)|enum|não|-|
|`Airline.Passengers[n].Phone`|Telefone do passageiro <br/> Ex.: 552121114700|string|Não|19|
|`Airline.Passengers[n].Email`|E-mail do passageiro|string|Não|60|
|`Airline.Passengers[n].LoyaltyMemberNumber`|Número de fidelidade do passageiro|string|Não|255|
|`Airline.Passengers[n].TicketNumber`|Número da passagem|string|Não|20|
|`Airline.Passengers[n].Legs[n].DepartureAirport`|Código do aeroporto de partida. Mais informações em [IATA 3-Letter Codes](http://www.nationsonline.org/oneworld/IATA_Codes/airport_code_list.htm){:target="_blank"}|string|Não|3|
|`Airline.Passengers[n].Legs[n].DepartureCountry`|Código do país do aeroporto de saída. Mais informações em [ISO 3-Digit Alpha Country Code](https://www.iso.org/obp/ui){:target="_blank"}|string|Não|3|
|`Airline.Passengers[n].Legs[n].ArrivalAirport`|Código do aeroporto de chegada. Mais informações em [IATA 3-Letter Codes](http://www.nationsonline.org/oneworld/IATA_Codes/airport_code_list.htm){:target="_blank"}|string|Não|3|
|`Airline.Passengers[n].Legs[n].ArrivalCountry`|Código do país do aeroporto de chegada. Mais informações em [ISO 3-Digit Alpha Country Code](https://www.iso.org/obp/ui){:target="_blank"}|string|Não|3|
|`Airline.Passengers[n].Legs[n].AirlineCode`|Código da companhia aérea|string|Não|3|
|`Airline.Passengers[n].Legs[n].DepartureDateTime`|Data e hora de partida <br/> Ex.: 2018-03-31 19:16:38 |datetime|Não|-|
|`Airline.Passengers[n].Legs[n].ClassOfService`|Classe de serviço|string|Não|30|
|`CustomConfiguration.MerchantWebsite`|Website da loja|string|Não|60|
|`MerchantDefinedData[n].Key`|Chave do campo definido junto ao provedor de antifraude <br/> [Tabela 35 - MerchantDefinedData (ACI Worldwide)](https://braspag.github.io/manual/antifraude#tabela-35-merchantdefineddata-(aci-worldwide))|int|Não|-|
|`MerchantDefinedData[n].Value`|Valor do campo definido junto ao provedor de antifraude <br/> [Tabela 35 - MerchantDefinedData (ACI Worldwide)](https://braspag.github.io/manual/antifraude#tabela-35-merchantdefineddata-(aci-worldwide))|var|Não|-|

### Resposta

``` json
{
   "TransactionId": "fdf8f357-a723-e811-80c3-0003ff21d83f",
   "Status": "Accept",
   "ProviderAnalysisResult": {
       "ProviderRequestId": "8a829449620619e801620b31d1c85d5a",
       "Result": {
           "ProviderCode": "000.000.000",
           "ProviderDescription": "Transaction succeeded"
       },
       "ResultDetails": {
           "CSITransactionLink": "https://csi-stage.redworldwide.com/index.red#transactiondetail/000548000001XAR20180309093717761",
           "ProviderStatus": "ACCEPT",
           "ProviderTransactionId": "381069636258",
           "ProviderResponseCode": "0150",
           "ProviderOrderId": "000548000001XAR20180309093717761"
       },
       "Ndc": "8a82941859d5969a0159db3f6ecc1418_60d2e8536e244db2bf04146872b00d38"
   },
   "Links": [
       {
           "Method": "GET",
           "Href": "http://localhost:1316/Analysis/v2/fdf8f357-a723-e811-80c3-0003ff21d83f",
           "Rel": "Self"
       }
   ]
}

```

**Parâmetros no cabeçalho (header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|
|`Status`|201 Created|

**Parâmetros no corpo (body)**

|Parâmetro|Descrição|Tipo|
|:-|:-|:-:|
|`TransactionId`|Id da transação no Antifraude Gateway Braspag|GUID|
|`Status`|Status da transação no Antifraude Gateway Braspag <br/> [Tabela 19 - Status](https://braspag.github.io/manual/antifraude#tabela-19-status)|enum|
|`ProviderAnalysisResult.ProviderRequestId`|Id do request da transação na ReDShield|string|
|`ProviderAnalysisResult.Result.ProviderCode`|Código de retorno da ReDShield|string|
|`ProviderAnalysisResult.Result.ProviderDescription`|Mensagem de retorno da ReDShield|string|
|`ProviderAnalysisResult.ResultDetails.CSITransactionLink`|Link para visualizar os detalhes da transação no portal CSI da ReDShield|string|
|`ProviderAnalysisResult.ResultDetails.ProviderStatus`|Status da transação na ReDShield <br/> [Tabela 20 - ProviderStatus](https://braspag.github.io/manual/antifraude#tabela-20-providerstatus)|enum|
|`ProviderAnalysisResult.ResultDetails.ProviderTransactionId`|Id da transação na ReDShield|string|
|`ProviderAnalysisResult.ResultDetails.ProviderOrderId`|Id do pedido na ReDShield|string|
|`ProviderAnalysisResult.Ndc`|Id único e exclusivo da requisição da ReDShield|string|

## Consultando uma transação na ACI Worldwide

### Requisição

<aside class="request"><span class="method get">GET</span> <span class="endpoint">analysis/v2/{Id}</span></aside>

``` json
{
  "TransactionId": "fdf8f357-a723-e811-80c3-0003ff21d83f",
   "Status": "Accept",
   "ProviderAnalysisResult": {
       "ProviderRequestId": "8a829449620619e801620b31d1c85d5a",
       "Result": {
           "ProviderCode": "000.000.000",
           "ProviderDescription": "Transaction succeeded"
       },
       "ResultDetails": {
           "CSITransactionLink": "https://csi-stage.redworldwide.com/index.red#transactiondetail/000548000001XAR20180309093717761",
           "ProviderStatus": "ACCEPT",
           "ProviderTransactionId": "381069636258",
           "ProviderResponseCode": "0150",
           "ProviderOrderId": "000548000001XAR20180309093717761"
       },
       "Ndc": "8a82941859d5969a0159db3f6ecc1418_60d2e8536e244db2bf04146872b00d38"
   },
   "Links": [
       {
           "Method": "GET",
           "Href": "http://localhost:1316/Analysis/v2/fdf8f357-a723-e811-80c3-0003ff21d83f",
           "Rel": "Self"
       }
   ],
  "MerchantOrderId": "4493d42c-8732-4b13-aadc-b07e89732c26",
  "TotalOrderAmount": 15000,
  "TransactionAmount": 14000,
  "Currency": "BRL",
  "Provider": "RedShield",
  "OrderDate": "2016-12-09 12:35:58.852",
  "BraspagTransactionId":"a3e08eb2-2144-4e41-85d4-61f1befc7a3b",
  "SplitingPaymentMethod": "None",
  "IsRetryTransaction": false,
  "Card": {
    "Number" : "4444555566667777",
    "Holder": "Holder Name",
    "ExpirationDate": "12/2023",
    "Cvv": "999",
    "Brand": "VISA",
    "EciThreeDSecure": "5"
  },
  "Billing": {
    "Street": "Rua Neturno",
    "Number": "12345",
    "Complement": "Sala 123",
    "Neighborhood": "Centro",
    "City": "Rio de Janeiro",
    "State": "RJ",
    "Country": "BR",
    "ZipCode": "20080123"
  },
  "Shipping": {
    "Street": "Rua Saturno",
    "Number": "30000",
    "Complement": "sl 123",
    "Neighborhood": "Centro",
    "City": "Rio de Janeiro",
    "State": "RJ",
    "Country": "BR",
    "ZipCode": "123456789",
    "Email": "emailentrega@dominio.com.br",
    "FirstName": "João",
    "MiddleName": "P",
    "LastName": "Silvao",
    "ShippingMethod": "SameDay",
    "Phone": "552121114700",
    "WorkPhone": "552121114721",
    "Mobile": "5521998765432",
    "Comment": "Em frente ao 322"
  },
  "Customer": {
    "MerchantCustomerId": "10050665740",
    "FirstName": "João",
    "MiddleName": "P",
    "LastName": "Silva",
    "BirthDate": "1983-10-01",
    "Gender": "Male",
    "Email": "emailcomprador@dominio.com.br",
    "Phone": "552121114700",
    "WorkPhone": "552121114721",
    "Mobile": "5521998765432",
    "Ip": "127.0.0.1",
    "BrowserFingerprint": "04003hQUMXGB0poNf94lis1ztuLYRFk+zJ17aP79a9O8mWOBmEnKs6ziAo94ggAtBvKEN6/FI8Vv2QMAyHLnc295s0Nn8akZzRJtHwsEilYx1P+NzuNQnyK6+7x2OpjJZkl4NlfPt7h9d96X/miNlYT65UIY2PeH7sUAh9vKxMn1nlPu2MJCSi12NBBoiZbfxP1Whlz5wlRFwWJi0FRulruXQQGCQaJkXU7GWWZGI8Ypycnf7F299GIR12G/cdkIMFbm6Yf0/pTJUUz1vNp0X2Zw8QydKgnOIDKXq4HnEqNOos1c6njJgQh/4vXJiqy0MXMQOThNipDmXv9I185O+yC2f3lLEO0Tay66NZEyiLNePemJKSIdwO9O5ZtntuUkG6NTqARuHStXXfwp8cyGF4MPWLuvNvEfRkJupBy3Z8hSEMEK7ZWd2T2HOihQxRh4qp+NANqYKBTl3v6fQJAEKikeSQVeBN8sQqAL0BZFaIMzbrnMivi6m6JRQUIdvEt+MbJEPFc0LjRycC5ApUmJO+Aoo9VKL1B8ftMSQ1iq1uTKn16ZOmDpzZrZhMPbH83aV0rfB2GDXcjpghm9klVFOw7EoYzV7IDBIIRtgqG9KZ+8NH/z6D+YNUMLEUuK1N2ddqKbS5cKs2hplVRjwSv7x8lMXWE7VDaOZWB8+sD1cMLQtEUC0znzxZ4bpRaiSy4dJLxuJpQYAFUrDlfSKRv/eHV3QiboXLuw9Lm6xVBK8ZvpD5d5olGQdc+NgsqjFnAHZUE+OENgY4kVU9wB84+POrI4MkoD4iHJ5a1QF8AZkZDFo1m1h9Bl+J2Ohr6MkBZq8DG5iVaunHfxUdHou5GL7lS1H7r+8ctfDXi8AfOPjzqyODJQ74Aiel35TKTOWG8pq1WO6yzJ1GNmMuMWZBamlGXoG/imnjwHY9HQtQzpGfcm0cR8X2Fd1ngNFGLDGZlWOX0jWtOwU6XVGT37JFD9W/cx4kzI+mPNi65X5WFPYlDG9N0Lbh5nOj3u3DXqRCiKCUrsEkMt8z9fxO9pLLGVQUKIYR2wTw53CiWK96FOpPevDWtH2XR0QkfOd02D73n81x6hEMCy0s3hRLn08Th9FlNHDMJBqLj+Tz8rG2TtNki3mJC7Ass1MT2qnKBI77n6vsQkAp59TfbZm/tBXwAoYdLJXge8F/numhd5AvQ+6I8ZHGJfdN3qWndvJ2I7s5Aeuzb8t9//eNsm73fIa05XreFsNyfOq1vG2COftC6EEsoJWe5h5Nwu1x6PIKuCaWxLY+npfWgM0dwJPmSgPx7TNM31LyVNS65m83pQ+qMTRH6GRVfg7HAcS5fnS/cjdbgHxEkRmgkRq1Qs48sbX9QC8nOTD0ntb6FcJyEOEOVzmJtDqimkzDq+SXR1/63AYe4LEj+ogRgN+Z8HAFhGFzd/m6snVviELfRqJ4LLQIk9Y/fzqnsF6I5OGxfdT2sxxK2Vokpi3jWhCcEknw7dYlHYpOnCHZO7QVgjQTngF2mzKf4GeOF4ECFsWTgLy6HFEitfauYJt1Xh1NfZZerBMwXLFzdhzoTQxGlcXc8lZIoEG1BLYv/ScICf8Ft9PEtpEa+j0cDSlU99UoH2xknwR1W9MRGc5I/euE63/IMJTqguZ3YcnJpjSVnAGSpyz/0gKjypJ3L86rHFRGXt0QbmaXtSl2UmmjI0p0LCCdx7McatCFEVI6FwPpPV0ZSMv/jM75eBid1X/lTV4XNzjowzR/iFlKYMzHZtVO9hCBPKlTwblRXNn4MlvNm/XeSRQ+Mr0YV5w5CL5Z/tGyzqnaLPj/kOVdyfj8r2m5Bcrz4g/ieUIo8qRFv2T2mET46ydqaxi27G4ZYHj7hbiaIqTOxWaE07qMCkJw==",
    "Status": "NEW"
  },
  "CartItems": [
    {
      "ProductName": "Mouse",
      "UnitPrice": "6500",
      "MerchantItemId": "4",
      "Sku": "abc123",
      "Quantity": 1,
      "OriginalPrice": "7000",
      "GiftMessage": "Feliz aniversário!",
      "Description": "Uma description do Mouse",
      "ShippingInstructions": "Proximo ao 546",
      "ShippingMethod": "SameDay",
      "ShippingTrackingNumber": "123456"
    },
    {
      "ProductName": "Teclado",
      "UnitPrice": "7500",
      "MerchantItemId": "3",
      "Sku": "abc456",
      "Quantity": 1,
      "OriginalPrice": "8000",
      "GiftMessage": "Parabéns!",
      "Description": "Uma description do Teclado",
      "ShippingInstructions": "Proximo ao 123",
      "ShippingMethod": "SameDay",
      "ShippingTrackingNumber": "987654"
    }
  ],
  "CustomConfiguration": {
    "MerchantWebsite": "www.test.com"
  },
  "MerchantDefinedData": [
    {
      "Key": "USER_DATA4",
      "Value": "Valor definido com o Provedor a ser enviado neste campo."
    },
    {
      "Key": "Segment",
      "Value": "8999"
    },
    {
      "Key": "MerchantId",
      "Value": "Seller123456"
    }
  ],
  "Airline": {
    "ThirdPartyBooking": "Y",
    "Bookingtype": "Corporate",
    "TicketDeliveryMethod": "Delivery",
    "BookingReferenceNumber": "L5W4NW",
    "Passengers": [
    {
        "FirstName": "Fulano",
        "MiddleName": "D",
        "LastName": "Tal",
        "PassengerType": "Adult",
        "Email": "email@mail.com",
        "Phone": "1234567890",
        "TicketNumber": "123541",
        "LoyaltyMemberNumber": "159753852",
        "Legs" : [
        {
            "ArrivalAirport": "AMS",
            "DepartureAirport": "GIG",
            "ArrivalCountry": "NLD",
            "DepartureCountry": "BRA",
            "AirlineCode": "KLM",
            "DepartureDateTime": "2018-01-09 18:00",
            "ClassOfService": "Standard"
        }]
    }]
  }
}
```

**Parâmetros no cabeçalho (header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|
|`Authorization`|Bearer {access_token}|
|`MerchantId`|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|
|`RequestId`|nnnnnnnn-nnnn-nnnn-nnnn-nnnnnnnnnnnn|

### Resposta

**Parâmetros no cabeçalho (header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|
|`Status`|200 OK|

**Parâmetros no corpo (body)**

|Parâmetro|Descrição|Tipo|
|:-|:-|:-:|
|`TransactionId`|Id da transação no Antifraude Gateway Braspag|GUID|
|`Status`|Status da transação no Antifraude Gateway Braspag <br/> [Tabela 20 - Status]({{ site.baseurl_root }}manual/antifraude#tabela-20-status)|enum|
|`ProviderAnalysisResult.ProviderRequestId`|Id do request da transação na ReDShield|string|
|`ProviderAnalysisResult.Result.ProviderCode`|Código de retorno da ReDShield|string|
|`ProviderAnalysisResult.Result.ProviderDescription`|Mensagem de retorno da ReDShield|string|
|`ProviderAnalysisResult.ResultDetails.CSITransactionLink`|Link para visualizar os detalhes da transação no portal CSI da ReDShield|string|
|`ProviderAnalysisResult.ResultDetails.ProviderStatus`|Status da transação na ReDShield <br/> [Tabela 21 - ProviderStatus]({{ site.baseurl_root }}manual/antifraude#tabela-21-providerstatus)|enum|
|`ProviderAnalysisResult.ResultDetails.ProviderTransactionId`|Id da transação na ReDShield|string|
|`ProviderAnalysisResult.ResultDetails.ProviderOrderId`|Id do pedido na ReDShield|string|
|`ProviderAnalysisResult.Ndc`|Id único e exclusivo da requisição da ReDShield|string|
|`MerchantOrderId`|Número do pedido da loja|string|
|`TotalOrderAmount`|Valor total do pedido em centavos <br/> Ex: 123456 = r$ 1.234,56|long|
|`TransactionAmount`|Valor da transação financeira em centavos <br/> Ex: 150000 = r$ 1.500,00|long|
|`Currency`|Moeda. Maiores informações em [ISO 4217 Currency Codes](https://www.iso.org/iso-4217-currency-codes.html){:target="_blank"}|enum|
|`Provider`|Provedor da solução de antifraude <br/> [Tabela 1 - Provider]({{ site.baseurl_root }}//manual/antifraude#tabela-1-provider)|enum|
|`OrderDate`|Data do pedido <br/> Ex.: 2016-12-09 19:16:38.155|datetime|
|`BraspagTransactionId`|Id da transação no Pagador da Braspag|GUID|
|`Tid`|Id da transação na adquirente <br/> Obs.: Caso você não possua integração com o Pagador Braspag, não terá como enviar o campo `BraspagTransactionId`, com isso é necessário o envio dos campos `Nsu`, `AuthorizationCode` e `SaleDate`, além deste em questão|string|
|`Nsu`|Número sequencial único da transação na adquirente <br/> Obs.: Caso você não possua integração com o Pagador Braspag, não terá como enviar o campo `BraspagTransactionId`, com isso é necessário o envio dos campos `Tid`, `AuthorizationCode` e `SaleDate`, além deste em questão|string|
|`AuthorizationCode`|Código de autorização da transação na adquirente <br/> Obs.: Caso você não possua integração com o Pagador Braspag, não terá como enviar o campo `BraspagTransactionId`, com isso é necessário o envio dos campos `Tid`, `Nsu` e `SaleDate`, além deste em questão|string|
|`SaleDate`|Data da autorização da transação da transação na adquirente <br/> Obs.: Caso você não possua integração com o Pagador Braspag, não terá como enviar o campo `BraspagTransactionId`, com isso é necessário o envio dos campos `Tid`, `Nsu` e `AuthorizationCode`, além deste em questão|datetime|
|`SplitingPaymentMethod`|Identifica se a autorização da transação é com um ou mais cartões ou com mais de um meio de pagamento <br/> [Tabela 2 - SplitingPaymentMethod]({{ site.baseurl_root }}manual/antifraude#tabela-2-splitingpaymentmethod)|enum|
|`IsRetryTransaction`|Retentativa de uma análise, e deverá ser enviado com valor igual a TRUE quando o código de retorno na primeira tentativa for igual a BP900|booleano|
|`Card.Number`|Número do cartão de crédito|string|
|`Card.Holder`|Nome do cartão de crédito|string|
|`Card.ExpirationDate`|Data de expiração do cartão de crédito <br/> Ex.: 01/2023|string|
|`Card.Cvv`|Código de segurança do cartão de crédito|string|
|`Card.Brand`|Bandeira do cartão de crédito <br/> [Tabela 3 - Card.Brand]({{ site.baseurl_root }}manual/antifraude#tabela-3-card.brand)|enum|
|`Card.EciThreeDSecure`|Código do ECI (Eletronic Commerce Indicator) de autenticação|string|
|`Card.Save`|Indica se os dados do cartão de crédito serão armazenados no Cartão Protegido|booleano|
|`Card.Token`|Identificador do cartão de crédito salvo no Cartão Protegido|GUID|
|`Card.Alias`|Alias (apelido) do cartão de crédito salvo no Cartão Protegido|string|
|`Billing.Street`|Logradouro do endereço de cobrança|string|
|`Billing.Number`|Número do endereço de cobrança|string|
|`Billing.Complement`|Complemento do endereço de cobrança|string|
|`Billing.Neighborhood`|Bairro do endereço de cobrança|string|
|`Billing.City`|Cidade do endereço de cobrança|string|
|`Billing.State`|Estado do endereço de cobrança|string|
|`Billing.Country`|País do endereço de cobrança. Mais informações em [ISO 2-Digit Alpha Country Code](https://www.iso.org/obp/ui)|string|
|`Billing.ZipCode`|Código postal do endereço de cobrança|string|
|`Shipping.Street`|Logradouro do endereço de entrega|string|
|`Shipping.Number`|Número do endereço de entrega|string|
|`Shipping.Complement`|Complemento do endereço de entrega|string|
|`Shipping.Neighborhood`|Bairro do endereço de entrega|string|
|`Shipping.City`|Cidade do endereço de entrega|string|
|`Shipping.State`|Estado do endereço de entrega|string|
|`Shipping.Country`|País do endereço de entrega. Mais informações em [ISO 2-Digit Alpha Country Code](https://www.iso.org/obp/ui){:target="_blank"}|string|
|`Shipping.ZipCode`|Código postal do endereço de entrega|string|
|`Shipping.Email`|E-mail do responsável a receber o produto no endereço de entrega|string|
|`Shipping.FirstName`|Primeiro nome do responsável a receber o produto no endereço de entrega|string|
|`Shipping.MiddleName`|Primeira letra do nome do meio do responsável a receber o produto no endereço de entrega|string|
|`Shipping.LastName`|Último do nome do responsável a receber o produto no endereço de entrega|string|
|`Shipping.Phone`|Número do telefone do responsável a receber o produto no endereço de entrega <br/> Ex.: 552121114700|string|
|`Shipping.WorkPhone`|Número do telefone de trabalho do responsável a receber o produto no endereço de entrega <br/> Ex.: 552121114701|string|
|`Shipping.Mobile`|Número do celular do responsável a receber o produto no endereço de entrega <br/> Ex.: 5521987654321|string|
|`Shipping.ShippingMethod`|Meio de entrega do pedido <br/> [Tabela 4 - ShippingMethod]({{ site.baseurl_root }}manual/antifraude#tabela-4-shippingmethod)|enum|
|`Shipping.Comment`|Referências do endereço de entrega|string|
|`Customer.MerchantCustomerId`|Número do documento de identificação do comprador, CPF ou CNPJ|string|
|`Customer.FirstName`|Primeiro nome do comprador|string|
|`Customer.MiddleName`|Primeira letra do nome do comprador|string|
|`Customer.LastName`|Último nome do comprador|string|
|`Customer.BirthDate`|Data de nascimento do comprador <br/> Ex.: 1983-10-01|data|
|`Customer.Gender`|Sexo do comprador <br/> [Tabela 6 - Customer.Gender]({{ site.baseurl_root }}//manual/antifraude#tabela-6-customer.gender)|string|
|`Customer.Email`|E-mail do comprador|string|
|`Customer.Ip`|Endereço de IP do comprador|string|
|`Customer.Phone`|Número do telefone do comprador <br/> Ex.: 552121114700|string|
|`Customer.WorkPhone`|Número do telefone do comprador <br/> Ex.: 552121114701|string|
|`Customer.Mobile`|Número do celular do comprador <br/> Ex.: 5521987654321|string|
|`Customer.Status`|Status do comprador na loja <br/> [Tabela 7 - Customer.Status]({{ site.baseurl_root }}manual/antifraude#tabela-7-customer.status)|string|
|`Customer.BrowserFingerPrint`|Impressão digital de dispositivos e geolocalização real do IP do comprador|string|
|`CartItem[n].ProductName`|Nome do produto|string|
|`CartItem[n].UnitPrice`|Preço unitário do produto <br/> Ex: 10950 = r$ 109,50|long|
|`CartItem[n].OriginalPrice`|Preço original do produto <br/> Ex: 11490 = r$ 114,90|long|
|`CartItem[n].MerchantItemId`|ID do produto na loja|string|
|`CartItem[n].Sku`|Sku do produto|string|
|`CartItem[n].Quantity`|Quantidade do produto|int|
|`CartItem[n].GiftMessage`|Mensagem de presente|string|
|`CartItem[n].Description`|Descrição do produto|string|
|`CartItem[n].ShippingInstructions`|Instruções de entrega do produto|string|
|`CartItem[n].ShippingMethod`|Meio de entrega do produto <br/> [Tabela 4 - ShippingMethod]({{ site.baseurl_root }}manual/antifraude#tabela-4-shippingmethod)|enum|
|`CartItem[n].ShippingTranckingNumber`|Número de rastreamento do produto|string|
|`Airline.ThirdPartyBooking`|Indica se a reserva foi agendada por terceiros, como por exemplo agências de turismo|booleano|
|`Airline.BookingType`|Tipo de agendamento da reserva|string|
|`Airline.TicketDeliveryMethod`|Tipo de entrega da passagem|string|
|`Airline.BookingReferenceNumber`|Número de referêcia da reserva|string|
|`Airline.Passengers[n].FirstName`|Primeiro nome do passageiro|string|
|`Airline.Passengers[n].MiddleName`|Nome do meio do passageiro|string|
|`Airline.Passengers[n].LastName`|Último nome do passageiro|string|
|`Airline.Passengers[n].PassengerType`|Tipo do passageiro <br/> [Tabela 9 - Airline.Passengers{n}.PassengerType]({{ site.baseurl_root }}manual/antifraude#tabela-9-airline.passengers[n].passengertype)|enum|
|`Airline.Passengers[n].Phone`|Telefone do passageiro <br/> Ex.: 552121114700|string|
|`Airline.Passengers[n].Email`|E-mail do passageiro|string|
|`Airline.Passengers[n].LoyaltyMemberNumber`|Número de fidelidade do passageiro|string|
|`Airline.Passengers[n].TicketNumber`|Número da passagem|string|
|`Airline.Passengers[n].Legs[n].DepartureAirport`|Código do aeroporto de partida. Mais informações em [IATA 3-Letter Codes](http://www.nationsonline.org/oneworld/IATA_Codes/airport_code_list.htm){:target="_blank"}|string|
|`Airline.Passengers[n].Legs[n].DepartureCountry`|Código do país do aeroporto de saída. Mais informações em [ISO 3-Digit Alpha Country Code](https://www.iso.org/obp/ui){:target="_blank"}|string|
|`Airline.Passengers[n].Legs[n].ArrivalAirport`|Código do aeroporto de chegada. Mais informações em [IATA 3-Letter Codes](http://www.nationsonline.org/oneworld/IATA_Codes/airport_code_list.htm){:target="_blank"}|string|
|`Airline.Passengers[n].Legs[n].ArrivalCountry`|Código do país do aeroporto de chegada. Mais informações em [ISO 3-Digit Alpha Country Code](https://www.iso.org/obp/ui){:target="_blank"}|string|
|`Airline.Passengers[n].Legs[n].AirlineCode`|Código da companhia aérea|string|
|`Airline.Passengers[n].Legs[n].DepartureDateTime`|Data e hora de partida <br/> Ex.: 2018-03-31 19:16:38 |datetime|
|`Airline.Passengers[n].Legs[n].ClassOfService`|Classe de serviço|string|
|`CustomConfiguration.MerchantWebsite`|Website da loja|string|
|`MerchantDefinedData[n].Key`|Chave do campo definido junto ao provedor de antifraude <br/> [Tabela 36 - MerchantDefinedData(ReDShield)]({{ site.baseurl_root }}manual/antifraude#tabela-36-merchantdefineddata-(redshield))|int|não|-|
|`MerchantDefinedData[n].Value`|Valor do campo definido junto ao provedor de antifraude <br/> [Tabela 36 - MerchantDefinedData(ReDShield)]({{ site.baseurl_root }}manual/antifraude#tabela-36-merchantdefineddata-(redshield))|var|não|-|

## Fingerprint com a ACI Worldwide

O Fingerprint é a identificação digital do dispositivo do comprador. Essa identificação é composta por uma série de dados coletados na página de checkout do site ou aplicativo.

### Integração com sua página de checkout (site)

#### Como funciona?

![Fluxo]({{ site.baseurl_root }}/images/braspag/af/fingerprint.png)

1. A página de checkout da loja envia os atributos do dispositivo do comprador para a Iovation, criando assim a *caixa preta*;
2. O lojista recebe a sequência de caracteres criptografados da Iovation e escreve o mesmo na página de checkout em um campo do tipo *hidden*;
3. O lojista envia para a Braspag, junto com os demais dados da transação a ser analisada, a *caixa preta*;
4. A Braspag recebe todos os dados, valida e envia para a ACI Worldwide;
5. A ACI Worldwide recebe todos os dados, envia a *caixa preta* para a Iovation descriptografar;
6. A ACI Worldwide recebe da Iovation os atributos do dispositivo do comprador.

#### Como configurar?

1. Inclua o javascript da Iovation em sua página de checkout;
2. Adicione parâmetros de configuração no JavaScript;
3. Crie um campo do tipo *hidden* em sua página para escrever a *caixa preta* nele e enviá-lo junto com os dados da transação a ser analisada.

<aside class="notice">Não realize cache do script, pois pode ocorrer de vários dispositivos sejam identificados como sendo o mesmo.</aside>

**Incluindo o JavaScript da Iovation**

Para incluir o JavaScript, adicione o seguinte elemento **&lt;script&gt;** na sua página de checkout.

Esta é a URL da versão do snare.js da Iovation: &lt;script type="text/javascript" src="https://mpsnare.iesnare.com/snare.js"&gt;&lt;/script&gt;

#### Parâmetros de configuração

|Parâmetro|Descrição|Default|
|:-|:-|:-|
|`io_install_flash`|Determina se será solicitado ao usuário a instalação do Flash ou atualização da versão|false|
|`io_flash_needs_handler`|Este parâmetro só terá validade se o parâmetro `io_install_flash` estiver configurado como TRUE, caso contrário não será executado <br/> É possível aqui customizar sua própria mensagem caso o Flash não esteja instalado <br/> Ex.: var `io_flash_needs_handler` = "Alert('Instalar Flash');"|-|
|`io_install_stm`|Determina se será solicitado ao usuário a instalação do Active X, que ajuda a coletar informações do hardware <br/> Este controle está disponível somente para o Internet Explorer, e caso o Active X já se encontre instalado, esta configuração não terá efeito|false|
|`io_exclude_stm`|Determina se o Active X deverá ser executado quando instalado <br/> É possível optar por desativar o controle para plataformas específicas <br/> Possíveis valores: <br/> 0 - executa em todas as plataformas <br/> 1 - não executa no Windows 9.x (incluindo as versões 3.1, 95, 98 e ME) <br/> 2 - não executa no Windows CE <br/> 4 - não executa no Windows XP (incluindo as versões NT, 2000, 2003 e 8) <br/> 8 - não executa no Windows Vista <br/> Obs.: Os valores são a combinação de somas dos valores acima, por exemplo: 12 - não executa no Windows XP (4) ou no Windows Vista (8)|15|
|`io_bbout_element_id`|Id do elemento HTML para preencher com a *caixa preta* <br/> Se o parâmetro `io_bb_callback` for definido, este não terá efeito|-|
|`io_enable_rip`|Determina se tentará coletar informações para obter o endereço IP real do comprador|true|
|`io_bb_callback`|Parâmetro para customizar a checagem da coleta da *caixa preta* foi concluída <br/> Ao utilizar, escrever a função conforme com a seguinte sintaxe: <br/> *io_callback(bb, complete)*, onde: <br/> bb - valor da caixa preta <br/> complete - valor booleano que indica que a coleta foi concluída|-|

<aside class="warning">IMPORTANTE: Os parâmetros de configuração devem ser colocados antes da chamada da tag acima; eles determinam como o JavaScript do iovation funcionará, e podem ocorrer erros caso sejam colocados antes da chamada do JavaScript.</aside>

**Exemplo**
![Exemplo HTML]({{ site.baseurl_root }}/images/braspag/af/exemplohtmlred.png)

### Integração em aplicativos mobile iOS e Android

**Baixando o SDK**
Se você ainda não baixou o SDK do iOS ou do Android, deve fazê-lo antes de continuar. Para isso acesse um dos links de acordo com o desejado.
* [Download Deviceprint SDK iOS](https://github.com/iovation/deviceprint-SDK-iOS)
* [Download Deviceprint SDK Android](https://github.com/iovation/deviceprint-SDK-Android)

**Sobre a integração**
Adicione o Iovation Mobile SDK aos seus aplicativos para coletar informações sobre os dispositivos dos compradores. Será gerada uma *caixa preta* que contém todas as informações do dispositivo disponíveis.

![Fluxo da coleta do fingerprint mobile]({{ site.baseurl_root }}/images/braspag/af/fingerprintmobile.png)

#### Integrando com aplicativos iOS

Arquivos e requisitos de integração do iOS
![Detalhes integração iOS]({{ site.baseurl_root }}/images/braspag/af/fingerprintios1.png)

Esta versão suporta iOS 5.1.1 ou superior nos seguintes dispositivos:
- iPhone 3GS e posterior
- iPod Touch 3ª geração ou posterior
- Todos os iPads

**Instalando o SDK no iOS**

1. Baixe e descompacte o SDK;

2. No Xcode, arraste *iovation.framework* na área de navegação do seu projeto
![Detalhes instalação SDK]({{ site.baseurl_root }}/images/braspag/af/fingerprintios2.png)

3. Na caixa de diálogo que aparece:
- Selecione *Copy items if needed* para copiar o framework para o diretório do projeto
- Marque a caixa de seleção para os destinos nos quais você planeja usar o framework
![Detalhes instalação SDK]({{ site.baseurl_root }}/images/braspag/af/fingerprintios3.png)

4. Clique em Finish

5. Adicione os frameworks a seguir ao destino da aplicação no XCode:
*ExternalAccessory.framework*. Se você verificar que o Wireless Accessory Configuration está ativado no Xcode 6 ou superior e não precisa, desativa e adicione novamente o ExternalAccessory.framework
*CoreTelephony.framework*
![Detalhes instalação SDK]({{ site.baseurl_root }}/images/braspag/af/fingerprintios4.png)

6. Opcionalmente, adicione esses frameworks se o seu aplicativo fizer uso deles:
*AdSupport.framework*. Se o seu aplicativo exibe anúncios
Obs.: Não incluir se o seu aplicativo não utilizar anúncios, pois a App Store rejeita aplicativos que incluem o framework mas não usam anúncios
*CoreLocation.framework*. Se o seu aplicativo usa monitoramento local
Obs.: Não incluir, a menos que seu aplicativo solicite permissão de geolocalização do usuário

**Usando a função +ioBegin**

A função *+ioBegin* coleta informações sobre o dispositivo e gera uma *caixa preta*. Esta *caixa preta* deverá ser enviada através do campo *Customer.BrowserFingerPrint* em conjunto com os outros dados para análise.

**Sintaxe**

> NSSstring * bbox = [iovation ioBegin]

**Valores de retorno**

> bbox - string que contem a *caixa preta*

<aside class="warning">IMPORTANTE: A *caixa preta* que retornou de *+ioBegin* nunca deve estar vazia. Uma *caixa preta* vazia indica que a proteção oferecida pelo sistema pode ter sido comprometida.</aside>

**Exemplo**
![Exemplo Código]({{ site.baseurl_root }}/images/braspag/af/exemplocodigo1.png)

#### Integrando com aplicativos Android

Arquivos e requisitos de integração do Android
![Detalhes]({{ site.baseurl_root }}/images/braspag/af/fingerprintandroid.png){: .left }{:title="Detalhes integração Android"}

<aside class="notice">Se as permissões listadas não são necessárias pelo aplicativo, os valores obtidos obtidos utilizando essas permissões serão ignorados. As permissões não são necessárias para obter uma *caixa preta*, mas ajudam a obter mais informações do dispositivo.</aside>

A versão 1.2.0 do Iovation Mobile SDK para Android suporta versões do Android 2.1 ou superior.

**Instalando o SDK no Android**

1. Baixe e descompacte o deviceprint-lib-1.2.0.aar;
2. Inicie o IDE de sua escolha;
3. No Eclipse e Maven, faça o deploy do arquivo de extensão *.aar* no repositório Maven local, usando o maven-deploy. Mais detalhes em: [Maven Guide](http://maven.apache.org/guides/mini/guide-3rd-party-jars-local.html){:target="_blank"};
4. No Android Studio, selecione *File -> New Module*. Expande *More Modules* e escolha *Import existing .jar or .aar package*;
5. Selecione o arquivo deviceprint-lib-1.2.0.aar, e clique em *Finish*;
6. Certifique-se de que o device-lib é uma dependência de compilação no arquivo build.gradle.

![Detalhes]({{ site.baseurl_root }}/images/braspag/af/fingerprintandroid1.png){: .left }{:title="Detalhes integração Android"}

**Usando a função ioBegin**

A função *ioBegin* coleta informações sobre o dispositivo e gera uma *caixa preta*. Esta *caixa preta* deverá ser enviada através do campo *Customer.BrowserFingerPrint* em conjunto com os outros dados para análise.

**Sintaxe*

> public static String ioBegin(Context context)

**Parâmetros*

> context - uma instância da classe *android.content.Context* usado para acessar informações sobre o dispositivo

**Valores de retorno**

> string que contem a *caixa preta*

**IMPORTANTE**
A *caixa preta* que retornou de *ioBegin* nunca deve estar vazio. Uma *caixa preta* vazia indica que contem apenas *0500* indica que a proteção oferecida pelo sistema pode ter sido comprometida.

**IMPORTANTE**
O arquivo *device-lib-1.2.0.aar* deverá ser empacotado com o aplicativo.

* Compilando o aplicativo de exemplo no Android Studio

**IMPORTANTE**
Se a opção para executar o módulo não aparecer, selecione *File -> Project Structure* e abra o painel *Modules*. A partir disso, defina na lista a versão do Android SDK.

![Exemplo Código]({{ site.baseurl_root }}/images/braspag/af/exemplocodigo2.png)

1. Baixe e descompacte o deviceprint-lib-1.2.0.aar;
2. No Android Studio, selecione *File -> Open* ou clique em *Open Project* através da opção *quick-start*;
3. No diretório em que você descompactou o *deviceprint-lib-1.2.0.aar*, abra diretório *android-studio-sample-app* do aplicativo de exemplo;
4. Abra o arquivo *DevicePrintSampleActivity*;
5. Com algumas configurações, o Android Studio pode detectar um Android Framework no projeto e não configurá-lo. Neste caso, abra o *Event Log* e clique em *Configure*;
6. Um pop-up irá abrir para você selecionar o Android Framework. Clique em *OK* para corrigir os erros;
7. No Android Studio, selecione *File -> New Module*. Expanda *More Modules* e escolha *Import existing .jar or .aar package*;
8. Selecione o arquivo deviceprint-lib-1.2.0.aar, e clique em *Finish*;
9. Certifique-se de que o device-lib é uma dependência de compilação no arquivo build.gradle <br/> ![Detalhes integração Android]({{ site.baseurl_root }}/images/braspag/af/fingerprintandroid1.png);
10. Abra a pasta DevicePrintSampleActivity;
11. Na opção de navegação do projeto, abra *src/main/java/com/iovation/mobile/android/sample/DevicePrintSampleActivity.java*;
12. Clique com o botão direito e selecione *Run DevicePrintSampleAct*;
13. Selecione um dispositivo físico conectado ou um Android virtual para executar o aplicativo;
14. O aplicativo irá compilar e executar.

O exemplo é simples, há um botão e ao clicar uma caixa de texto é preenchida com a *caixa preta*. Para obter um exemplo mais rico, consulte o aplicativo de exemplo do Android Studio incluído no SDK.

# Integração com a ClearSale

A integração com a ClearSale é composta de duas etapas, a [configuração do fingerprint](https://braspag.github.io/manual/antifraude#fingerprint-com-a-clearsale) e a [requisição de análise de fraude](https://braspag.github.io/manual/antifraude#analisando-uma-transa%C3%A7%C3%A3o-na-clearsale).

## Fingerprint com a ClearSale

O Fingerprint é a identificação digital do dispositivo do comprador. Essa identificação é composta por uma série de dados coletados na página de checkout do site ou aplicativo como:

* Localização precisa (quando habilitada permissão pelo usuário);
* Identificadores de publicidade do dispositivo (quando habilitada permissão pelo usuário);
* Características físicas do dispositivo (tela, bateria, teclado, espaço livre em disco, modelo, nome do dispositivo);
* Características de software (versão, idioma, build, controle parental);
* Informações de rede (conexões, IP);
* Operadora do SimCard.<br/>
<br/>

<aside class="notice">IMPORTANTE: Para atender aos requisitos da Lei Geral de Proteção de Dados (LGPD), inclua a informação sobre coleta de dados do dispositivo do comprador na política de cookies do seu e-commerce.</aside>

**Fluxo do Fingerprint com a ClearSale**

Para análises via ClearSale, o Fingerprint é criado antes da requisição de análise de fraude por meio de um script em JavaScript.

A criação do Fingerprint acontece separada da requisição de análise de fraude.

Veja a representação do fluxo de criação do Fingerprint e requisição de análise de fraude:

![Fluxo Fingerprint Clearsale]({{ site.baseurl_root }}/images/braspag/af/fingerprint-clearsale.png)
 
**Etapa de criação do Fingerprint**

**1.** O comprador preenche os dados solicitados na página de checkout da loja (website ou aplicativo);

**2.** A página de checkout da loja, já configurada com o código do Fingerprint, coleta os dados do comprador e executa o script de identificação do dispositivo (criação do Fingerprint).

**Etapa de análise de fraude**

**3.** A loja envia a requisição de análise de fraude com o campo `Customer.BrowserFingerprint` (com o valor do `session_id`) para o Antifraude Gateway;

**4.** O Antifraude Gateway valida a requisição e solicita a análise de fraude para a ClearSale;

**5.** A ClearSale realiza a análise de fraude levando em conta o Fingerprint do dispositivo e envia a recomendação de aceitar, rejeitar ou revisar* a transação para o Antifraude Gateway;

**6.** O Antifraude Gateway retorna o resultado da análise de fraude para a loja;

**7.** A loja retorna o status da transação (aprovada ou não aprovada) ao comprador.

> *Variável conforme o pacote contratado.

### Onde enviar o Fingerprint?

Na requisição de análise de fraude com a ClearSale, envie o valor do `session_id` no parâmetro `Customer.BrowserFingerprint`.

### Como configurar o Fingerprint na ClearSale?

O Fingerprint consiste na implementação de um script na sua página de checkout (front-end), na parte onde o comprador preenche os dados cadastrais.

#### Variáveis do Fingerprint

A tabela a seguir apresenta as variáveis para configuração do Fingerprint com a ClearSale.

|VARIÁVEL|DESCRIÇÃO|TIPO|TAMANHO|OBRIGATÓRIO|
|---|---|---|---|---|
|`session_id`|Identifica a sessão do usuário no website e aplicativo. Pode ser criado pela loja ou pela ClearSale.<br>O valor do `session_id` deve ser submetido no campo `Customer.BrowserFingerprint`.|GUID (recomendado) |6 a 128 caracteres|Sim|
|`seu_appkey`|Identifica a loja para criação do fingerprint dos dispositivos dos compradores. Fornecido pela Cielo.|string|-|Sim|

> O `session_id` pode aparecer na documentação da ClearSale como `SessionID` ou `SessionId`.

#### Configurando o Fingerprint na ClearSale – Web

Você deverá inserir um script (JavaScript) no código front-end da sua página de checkout. Esse script tem o parâmetro `session_id`, que é o identificador da sessão do usuário no website ou aplicativo.

> Para monitorar caso o usuário faça o bloqueio do script, insira a tag `noscript`. Saiba mais em: [Monitoramento Bloqueio de Script](https://api.clearsale.com.br/docs/behavior-analytics/sdk/browser#implementation-script){:target="_blank"}.

Há três formas de incluir o script na sua página de checkout. A diferença está na criação do `session_id`:

* `session_id` criado pela loja;
* `session_id` criado pelo Google Tag Manager;
* `session_id` criado pela ClearSale (recomendado).

> Saiba mais detalhes na documentação [Behavior Analytics](https://api.clearsale.com.br/docs/behavior-analytics/sdk/browser#inclusion-page){:target="_blank"}.
<br>
> Na requisição de análise de fraude, envie no campo `Customer.BrowserFingerprint` apenas o valor do `session_id`.

#### Configurando o Fingerprint na Clearsale – Mobile

O `session_id` pode ser criado pela loja ou é possível solicitar a criação pela ClearSale pelo método `generateSessionID()`.

Consulte as instruções para cada sistema operacional:

* [Android](https://api.clearsale.com.br/docs/behavior-analytics/sdk/android/latest#introduction-section){:target="_blank"};
* [iOS](https://api.clearsale.com.br/docs/behavior-analytics/sdk/ios/latest#introduction-section){:target="_blank"}.

## Analisando uma transação na ClearSale

### Requisição

<aside class="request"><span class="method post">POST</span> <span class="endpoint">analysis/v2/</span></aside>

``` json
{
    "TotalOrderAmount": 46000,
    "TransactionAmount": 45500,
    "Provider": "ClearSale",
    "Currency": "BRL",
    "OrderDate": "2023-09-26T08:44:30.1945689",
    "BraspagTransactionId": "1cbdd27f-80e4-47a6-9921-59a24989536f",
    "Card": {
        "Number": "4000021231111111",
        "Holder": "Guilherme Silva",
        "ExpirationDate": "08/2033",
        "Brand": "Visa"
    },
    "Billing": {
        "Street": "Alameda Xingu",
        "Number": "512",
        "Complement": "21 andar",
        "Neighborhood": "Alphaville",
        "City": "Barueri",
        "State": "SP",
        "Country": "BR",
        "ZipCode": "06455030"
    },
    "Shipping": {
        "DocumentType": "Cpf",
        "DocumentNumber": "99988877711",
        "Street": "Alameda Xingu",
        "Number": "512",
        "Complement": "21 andar",
        "Neighborhood": "Alphaville",
        "City": "Barueri",
        "State": "SP",
        "Country": "BR",
        "ZipCode": "06455030",
        "Email": "nome@email.com.br",
        "FirstName": "Nome",
        "LastName": "Comprador",
        "Phone": "+55 11 5555-1001",
        "WorkPhone": "+55 11 5555-1002",
        "Mobile": "+55 11 5555-1003",
        "ShippingMethod": 0
    },
    "Customer": {
        "MerchantCustomerId": "11111111111",
        "DocumentType": "Cpf",
        "FirstName": "Bruno",
        "LastName": "Silva",
        "BirthDate": "1996-11-14",
        "Email": "homolog@cielo.com.br",
        "Ip": "127.0.0.1",
        "Phone": "+55 21 5555-1004",
        "Mobile": "+55 21 5555-1005",
        "WorkPhone": "+55 21 5555-1006",
        "BrowserFingerprint": "MzE5MjAzODg0NA=="
    },
    "Airline": {
        "Passengers": [
            {
                "FirstName": "Bruno",
                "LastName": "Silva",
                "PassengerId": "99988877714",
                "Legs": [
                    {
                        "ArrivalAirport": "CGH",
                        "DepartureAirport": "SDU",
                        "DepartureDateTime": "2023-10-09T18:30:00",
                        "Boarding": "2023-10-09T18:45:00",
                        "Arriving": "2023-10-09T20:00:00"
                    }
                ]
            },
            {
                "FirstName": "Guilherme",
                "LastName": "Silva",
                "PassengerId": "99988877714",
                "Legs": [
                    {
                        "ArrivalAirport": "CGH",
                        "DepartureAirport": "SDU",
                        "DepartureDateTime": "2023-10-09T18:30:00",
                        "Boarding": "2023-10-09T18:45:00",
                        "Arriving": "2023-10-09T20:00:00"
                    }
                ]
            }
        ]
    },
    "PaymentType": "CreditCard",
    "NumberOfInstallments": 1
}
```

> **Atenção**: Você só deve enviar o campo `BraspagTransactionId` se o seu fluxo for `AuthorizeFirst` e estiver usando o Pagador Braspag. O campo `BraspagTransactionId` é o identificador da transação no Pagador. Saiba mais na [documentação do Pagador](https://braspag.github.io//manual/braspag-pagador#pagamentos-com-an%C3%A1lise-de-fraude){:target="_blank"}.

**Parâmetros no cabeçalho (Header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|
|`Authorization`|Bearer {access_token}|
|`MerchantId`|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|
|`RequestId`|nnnnnnnn-nnnn-nnnn-nnnn-nnnnnnnnnnnn|

**Parâmetros no corpo (body)**

| Parâmetro | Descrição  | Tipo     | Tamanho | Obrigatório |
|-----------|------------|----------|---------|-------------|
| `TotalOrderAmount`                                | Valor total dos itens.                                                                                               | long     | -       | Sim           |
| `TransactionAmount`                               | Valor cobrado neste pagamento.                                                                                        | long     | -       | Não           |
|`Provider`| Provedor da solução Antifraude Gateway. Neste caso, usar "ClearSale". [Tabela 1 - Provider](https://braspag.github.io/manual/antifraude#tabela-1-provider)|enum|-|Sim|
|`Currency`|Moeda. Mais informações em [ISO 4217 Currency Codes](https://www.iso.org/iso-4217-currency-codes.html){:target="_blank"}|string|3|Sim|
|`BraspagTransactionId`|Id da transação no Pagador da Braspag. Obs.: Você pode enviar esse campo se o seu fluxo de análise de fraude é AuthorizeFirst, no qual a autorização acontece primeiro. Se você não tem integração com o Pagador Braspag, você pode enviar os campos `Tdi`,`Nsu`, `AuthorizationCode` e `SaleDate` em vez do campo `BraspagTransactionId`|GUID|Não|-|
| `Customer.BrowserFingerprint`                     | Identificador único da sessão do usuário.                                                                             | string   | 128     | Sim           |
| `OrderDate`                                       | Data do pedido (se não enviado retornaremos a data da requisição).                                       | datetime | 0       | Não           |
| `NumberOfInstallments`                            | Quantidade de parcelas.                                                                                               | int  | -       | Não           |
| `Customer.Ip`                                     | IP do comprador do pedido.                                                                                                         | string   | 50      | Não           |
| `Customer.DocumentType`                           | 1 = Pessoa física<br>2 = Pessoa jurídica.                                                                                | int  | -       | Sim           |
| `Customer.MerchantCustomerId`                     | CPF ou CNPJ.                                                                                                          | string   | 100     | Sim           |
| `Customer.FirstName`                              | Primeiro nome do comprador.                                                                                             | string   | 60      | Sim           |
| `Customer.LastName`                               | Sobrenome do comprador.                                                                                                 | string   | 60      | Sim           |
| `Customer.BirthDate`                              | Data de nascimento do comprador.                                                                                                   | datetime | 0       | Não           |
| `Customer.Email`                                  | E-mail do comprador.                                                                                                      | string   | 150     | Sim           |
| `Billing.Street`                                  | Nome do logradouro do endereço de cobrança.                                                                                                   | string   | 200     | Sim           |
| `Billing.Number`                                  | Número do endereço de cobrança.                                                                                                   | string   | 15      | Sim           |
| `Billing.Complement`                              | Complemento do endereço de cobrança.                                                                                              | string   | 250     | Não           |
| `Billing.Neighborhood`                            | Bairro do endereço de cobrança.                                                                                                   | string   | 150     | Sim           |
| `Billing.City`                                    | Cidade do endereço de cobrança.                                                                                                   | string   | 150     | Sim           |
| `Billing.State`                                   | Sigla do estado do endereço de cobrança - UF.                                                                                     | string   | 2       | Sim           |
| `Billing.Country`                                 | País do endereço de cobrança.                                                                                                     | string   | 150     | Não           |
| `Billing.ZipCode`                                 | CEP do endereço de cobrança.                                                                                                      | string   | 10      | Sim           |
| `Customer.Phone`                                  | Telefone residencial - Fomato +DDI DDD NNNNNNNN Exemplo: +55 11 33333333. **É obrigatório enviar pelo menos um telefone**. | string   | 20      | Não           |
| `Customer.WorkPhone`                              | Telefone comercial  - Fomato +DDI DDD NNNNNNNN Exemplo: +55 11 33333333. **É obrigatório enviar pelo menos um telefone**.  | string   | 20      | Não           |
| `Customer.Mobile`                                 | Celular - Fomato +DDI DDD NNNNNNNNN Exemplo: +55 11 999999999. **É obrigatório enviar pelo menos um telefone**. | string   | 20      | Não           |
| `Shipping.DocumentType`                           | 1 = Pessoa Física<br> 2 = Pessoa Jurídica.                                                                                | int  | -       | Sim           |
| `Shipping.DocumentNumber`                         | CPF ou CNPJ.                                                                                                          | string   | 100     | Sim           |
| `Shipping.FirstName`                              | Primeiro nome do destinatário.                                                                                        | string   | 60      | S           |
| `Shipping.LastName`                               | Sobrenome do destinatário.                                                                                            | string   | 60      | Sim           |
| `Shipping.Email`                                  | E-mail.                                                                                                                | string   | 150     | Não           |
| `Shipping.Street`                                 | Nome do logradouro.                                                                                                   | string   | 200     | Sim           |
| `Shipping.Number`                                 | Número do endereço.                                                                                                   | string   | 15      | Sim           |
| `Shipping.Complement`                             | Complemento do endereço.                                                                                              | string   | 250     | Não           |
| `Shipping.Neighborhood`                           | Bairro do endereço.                                                                                                   | string   | 150     | Sim           |
| `Shipping.City`                                   | Cidade do endereço.                                                                                                   | string   | 150     | Sim           |
| `Shipping.State`                                  | Sigla do estado do endereço - UF.  | string   | 2       | Sim           |
| `Shipping.Country`                                | País do endereço.                                                                                                     | string   | 150     | Não           |
| `Shipping.ZipCode`                                | CEP do endereço.                                                                                                      | string   | 10      | Sim           |
| `Shipping.Phone`                                  | Telefone residencial - Fomato +DDI DDD NNNNNNNN Exemplo: +55 11 33333333. **É obrigatório enviar pelo menos um telefone**. | string   | 20      | Não           |
| `Shipping.workPhone`                              | Telefone comercial  - Fomato +DDI DDD NNNNNNNN Exemplo: +55 11 33333333. **É obrigatório enviar pelo menos um telefone**.  | string   | 20      | Não           |
| `Shipping.Mobile`                                 | Celular - Fomato +DDI DDD NNNNNNNNN Exemplo: +55 11 999999999. **É obrigatório enviar pelo menos um telefone**.            | string   | 20      | Não           |
| `PaymentType`                                     | Tipo de Pagamento:<br>CreditCard = 1<br>DebitCard = 2   | int  | -       | Sim           |
| `Card.Number`                                     | Número do cartão.   | string   | 19      | Sim           |
| `Card.Brand`  | Bandeira do cartão de crédito. Veja mais em [Tabela 3 - Card.Brand](https://braspag.github.io/manual/antifraude#tabela-3-card.brand) |  enum    | -        | Não       |
| `Card.ExpirationDate`                             | Data da expiração.                                                                                                    | string   | 50      | Não           |
| `Card.Holder`                                     | Nome de cobrança.                                                                                                     | string   | 150     | Sim           |
| `CartItem[].Sku`                                  | Código do produto.                                                                                                    | string   | 50      | Não           |
| `CartItem[].ProductName`                          | Nome do produto.                                                                                                      | string   | 150     | Sim           |
| `CartItem[].UnitPrice`                            | Preço unitário do produto, em centavos.                                      | long     | -       | Não           |
| `CartItem[].Quantity`                             | Quantidade de produtos.                                                                                                           | int  | 0       | Não           |
| `CartItem[].Category`  | Categoria do produto. VEja mais em [Tabela 36 - CartItem{n}.Category](https://braspag.github.io/manual/antifraude#tabela-36-cartitem[n].category)               | enum    | 0        | Não       |             |
| `Airline[].Passengers[].FirstName`                | Primeiro nome do passageiro. **É obrigatório enviar o bloco `Airline` quando o segmento for companhia aérea**. | string   | 60      | Condicional*           |
| `Airline[].Passengers[].LastName`                 | Sobrenome do passageiro **É obrigatório enviar o bloco `Airline` quando o segmento for companhia aérea**.| string   | 60      | Condicional*           |
| `Airline[].Passengers[].PassengerId`|Identificador do passageiro para quem a passagem foi emitida.|string|32|Condicional*|
| `Airline[].Passengers[].Legs[].DepartureDateTime` | Data do voo.   | datetime | 0       | Condicional*           |
| `Airline[].Passengers[].Legs[].DepartureAirport`  | Aeroporto de origem.  | string   | 5       | Condicional*           |
| `Airline[].Passengers[].Legs[].ArrivalAirport`    | Aeroporto de destino.   | string   | 5       | Condicional*           |
| `Airline[].Passengers[].Legs[].Boarding`          | Data de embarque.  | datetime | 0       | Condicional*           |
| `Airline[].Passengers[].Legs[].Arriving`          | Data de desembarque.  | datetime | 0       | Condicional*           |

***É obrigatório enviar o bloco `Airline` quando o segmento for companhia aérea**.

### Resposta

``` json
{
    "TransactionId": "79c92335-cbd7-4313-1842-08dbbaaa0d60",
    "Status": "Accept",
    "ProviderAnalysisResult": {
        "ProviderTransactionId": "79C92335-CBD7-4313-1842-08DBBAAA0D60",
        "ProviderStatus": "Accept",
        "ProviderCode": 1,
        "Score": "99.99"
    },
    "TotalOrderAmount": 46000,
    "TransactionAmount": 45500,
    "Currency": "BRL",
    "Provider": "ClearSale",
    "OrderDate": "2023-09-26T08:44:30.1945689",
    "BraspagTransactionId": "1cbdd27f-80e4-47a6-9921-59a24989536f",
    "Card": {
        "Number": "400002******1111",
        "Holder": "Guilherme Silva",
        "ExpirationDate": "08/2033",
        "Cvv": "***",
        "Brand": "Visa"
    },
    "Billing": {
        "Street": "Alameda Xingu",
        "Number": "512",
        "Complement": "21 andar",
        "Neighborhood": "Alphaville",
        "City": "Barueri",
        "State": "SP",
        "Country": "BR",
        "ZipCode": "06455030"
    },
    "Shipping": {
        "DocumentType": "Cpf",
        "DocumentNumber": "99988877714",
        "Street": "Alameda Xingu",
        "Number": "512",
        "Complement": "21 andar",
        "Neighborhood": "Alphaville",
        "City": "Barueri",
        "State": "SP",
        "Country": "BR",
        "ZipCode": "06455030",
        "Email": "nome@email.com",
        "FirstName": "Nome",
        "LastName": "Comprador",
        "Phone": "+55 21 5555-1001",
        "WorkPhone": "+55 21 5555-1002",
        "Mobile": "+55 21 5555-1003",
        "ShippingMethod": "Undefined"
    },
    "Customer": {
        "MerchantCustomerId": "11111111111",
        "DocumentType": "Cpf",
        "FirstName": "Bruno",
        "LastName": "Silva",
        "FullName": "Bruno Silva",
        "BirthDate": "1996-11-14T00:00:00",
        "Email": "nome@email.com.br",
        "Ip": "127.0.0.1",
        "Phone": "+55 21 5555-1004",
        "Mobile": "+55 21 5555-1005",
        "WorkPhone": "+55 21 5555-1006",
        "BrowserFingerprint": "MzE5MjAzODg0NA=="
    },
    "Airline": {
        "JourneyType": 0,
        "Passengers": [
            {
                "FirstName": "Bruno",
                "LastName": "Silva",
                "PassengerId": "99988877714",
                "Status": "Undefined",
                "PassengerType": "Undefined",
                "Legs": [
                    {
                        "ArrivalAirport": "CGH",
                        "DepartureAirport": "SDU",
                        "DepartureDateTime": "2023-10-09T18:30:00",
                        "Boarding": "2023-10-09T18:45:00",
                        "Arriving": "2023-10-09T20:00:00"
                    }
                ]
            },
            {
                "FirstName": "Guilherme",
                "LastName": "Silva",
                "PassengerId": "99988877714",
                "Status": "Undefined",
                "PassengerType": "Undefined",
                "Legs": [
                    {
                        "ArrivalAirport": "CGH",
                        "DepartureAirport": "SDU",
                        "DepartureDateTime": "2023-10-09T18:30:00",
                        "Boarding": "2023-10-09T18:45:00",
                        "Arriving": "2023-10-09T20:00:00"
                    }
                ]
            }
        ]
    },
    "PaymentType": "CreditCard",
    "NumberOfInstallments": 1
}
    "Links": [
        {
            "Method": "GET",
            "Href": "http://risksandbox.braspag.com.br/Analysis/v2/79c92335-cbd7-4313-1842-08dbbaaa0d60",
            "Rel": "Self"
        }
    ],
}
```

**Parâmetros no cabeçalho (header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|
|`Status`|201 Created|

**Parâmetros no corpo (body)**

|Parâmetro|Descrição|Tipo|
|:-|:-|:-:|
|`TransactionId`|Id da transação no Antifraude Gateway Braspag.|GUID|
|`Status`|Status da transação no Antifraude Gateway Braspag <br/> [Tabela 19 - Status](https://braspag.github.io/manual/antifraude#tabela-19-status).|enum|
|`ProviderAnalysisResult.ProviderTransactionId`|Id da transação na ClearSale.|string|
|`ProviderAnalysisResult.ProviderStatus`|Status da transação na ClearSale. <br/> [Tabela 20 - ProviderStatus](https://braspag.github.io/manual/antifraude#tabela-20-providerstatus)|enum|
|`ProviderAnalysisResult.ProviderCode`|Código de retorno da ClearSale. <br/> [Tabela 21 - ProviderAnalysisResult.ProviderCode](https://braspag.github.io/manual/antifraude#tabela-21-provideranalysisresult.providercode)|int|
|`ProviderAnalysisResult.Score`| Score total calculado para o pedido. |number|

# Indicando erros de integração

## Erro da análise de fraude

Em caso de erro na requisição de análise de fraude, a resposta irá retornar a mensagem informando que a requisição é inválida, a relação de campos inválidos e a qual limite de tamanho não foi respeitado. Veja a seguir um exemplo de resposta de erro:

``` json
{
  "Message": "The request is invalid.",
  "ModelState": {
    "request.Customer.Gender": [
      "Error converting value \"M\" to type 'Antifraude.Domain.Enums.GenderType'. Path 'Customer.Gender', line 51, position 17."
    ],
    "FraudAnalysisRequestError": [
      "The Card.EciThreeDSecure lenght is gratter than 1",
      "The Shipping.Complement lenght is gratter than 14",
      "The Shipping.MiddleName lenght is gratter than 1",
      "The Customer.MerchantCustomerId lenght is gratter than 16",
      "The Customer.MiddleName lenght is gratter than 1"
    ]
  }
}
```

**Parâmetros no cabeçalho (header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|
|`Status`|400 Bad Request|

**Parâmetros no corpo (body)**

|Parâmetro|Descrição|
|:-|:-|
|`Message`|Mensagem informando que a requisição é inválida.|
|`ModelState`|Coleção que conterá mensagens com os campos que não estejam de acordo com o tipo ou domínio conforme especificado no manual.|
|`FraudAnalysisRequestError`|Coleção que conterá mensagens com os campos que não estejam de acordo com o tamanho especificado no manual.|

## Erro na consulta de uma transação

Ao consultar uma transação usando `Id` inexistente, a resposta será como o exemplo a seguir:

**Parâmetros no cabeçalho (header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|
|`Status`|404 Not Found|

# Post de Notificação

Para saber o status de uma transação, recomendamos que você configure o Post de Notificação para receber a notificação de mudança de status e também faça a consulta por sondagem para visualizar o status atual.

O Post de Notificação é um webhook que envia uma notificação para a URL cadastrada pela sua loja, caso haja alguma alteração de status na transação:

* De *Review* (em revisão) para *Accept* (aceita) ou *Reject* (rejeitada);
* De *Accept* (aceita) para *Reject* (rejeitada).
<br/>
<br/>
**Importante:**
<br/>
<br/>
* No processo de onboarding, solicite o cadastramento da URL de mudança de status da sua loja ao time de Implantação da Braspag;
* A URL cadastrada pela loja para receber a notificação da mudança de status deverá retornar o código HTTP 200 (OK), indicando que a mensagem foi recebida e processada com sucesso pelo servidor da loja. Caso contrário, serão realizadas mais 3 tentativas de envio;
* A URL de mudança de status somente pode utilizar a porta 80 (padrão para HTTP) ou a porta 443 (padrão para HTTPS). Recomendamos que a loja trabalhe sempre com SSL para esta URL, ou seja, sempre HTTPS.
<br/>
<br/>
> **Após receber a notificação de mudança de status, faça uma consulta** enviando o Id da transação recebido para obter o novo status da transação:<br/>
> <br/>
> **[Consulta na Cybersource](https://braspag.github.io//manual/antifraude#consultando-uma-transa%C3%A7%C3%A3o-cybersource)**<br/>
> **[Consulta na ACI Worldwide](https://braspag.github.io//manual/antifraude#consultando-uma-transa%C3%A7%C3%A3o-na-aci-worldwide)**

## Requisição

<aside class="request"><span class="method post">POST</span> <span class="endpoint">{url loja}</span></aside>

``` json
{  
   "Id":"tttttttt-tttt-tttt-tttt-tttttttttttt"
}
```

**Parâmetros no Cabeçalho (Header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|

**Parâmetros no Corpo (Body)**

|Parâmetro|Descrição|Tipo|
|:-|:-|:-:|
|`Id`|Id da transação no Antifraude Gateway Braspag|guid|

## Resposta

**Parâmetros no cabeçalho (header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|
|`Status`|200 OK|

# Associar transação

A associação de uma transação do Pagador Braspag ou de outra solução de autorização à uma transação do Antifraude Gateway Braspag pode ser feita quando o fluxo de análise de fraude é o **AuthorizeFirst**.

> Você deverá realizar esta chamada quando: <br/>
> 1. Realiza a análise de fraude através do Antifraude Gateway Braspag <br/>
> 2. Realiza a autorização através do Pagador Braspag ou outra solução de autorização.

Para saber mais sobre o modelo **AuthorizeFirst** de análise de fraude, em que a análise de risco acontece depois da autorização da transação, consulte o [Manual do Pagador](https://braspag.github.io//manual/braspag-pagador#pagamentos-com-an%C3%A1lise-de-fraude){:target="_blank"}.

## Antifraude e Pagador

### Requisição

<aside class="request"><span class="method patch">PATCH</span> <span class="endpoint">transaction/{id}</span></aside>

``` json
{
    "BraspagTransactionId": "a3e08eb2-2144-4e41-85d4-61f1befc7a3b"
}
```

**Parâmetros no cabeçalho (header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|
|`Authorization`|Bearer {access_token}|
|`MerchantId`|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|
|`RequestId`|nnnnnnnn-nnnn-nnnn-nnnn-nnnnnnnnnnnn|

**Parâmetros no corpo (body)**

|Parâmetro|Descrição|Tipo|Obrigatório|Tamanho|
|:-|:-|:-:|:-:|-:|
|`BraspagTransactionId`|Id da transação no Pagador Braspag|guid|sim|-|

### Resposta

A resposta sempre irá apresentar no header o `Content-Type` com o valor "application/json" e o `Status`. Veja a seguir o status possível para cada cenário:

|Status|Descrição|
|---|---|
|200 OK | A transação do Pagador foi associada corretamente com a transação do Antifraude Gateway.|
|400 Bad Request| A transação do Pagador não foi informada na requisição.|
|404 Not Found|A transação do Antifraude Gateway não foi encontrada na base de dados.|
|409 Conflict|A transação do Pagador já está associada à outra transação do Antifraude Gateway.|

## Antifraude e outra solução de autorização

### Requisição

<aside class="request"><span class="method put">PUT</span> <span class="endpoint">transaction/{id}</span></aside>

``` json
{
    "Tid": "12345678910111216AB8",
    "Nsu": "951852",
    "AuthorizationCode":"T12345",
    "SaleDate": "2016-12-09 10:01:55.662"
}
```

**Parâmetros no cabeçalho (header)**

|Key|Value|
|:-|:-|
|`Content-Type`|application/json|
|`Authorization`|Bearer {access_token}|
|`MerchantId`|xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx|
|`RequestId`|nnnnnnnn-nnnn-nnnn-nnnn-nnnnnnnnnnnn|

**Parâmetros no corpo (body)**

|Parâmetro|Descrição|Tipo|Obrigatório|Tamanho|
|:-|:-|:-:|:-:|-:|
|`Tid`|Id da transação na adquirente.|string|sim|20|
|`Nsu`|Número sequencial único da transação na adquirente.|string|sim|10|
|`AuthorizationCode`|Código de autorização da transação na adquirente.|string|sim|10|
|`SaleDate`|Data da autorização da transação da transação na adquirente.|datetime|sim|-|

### Resposta

A resposta sempre irá apresentar no header o `Content-Type` com o valor "application/json" e o `Status`. Veja a seguir o status possível para cada cenário:

|Status|Descrição|
|---|---|
|200 OK | A transação da outra solução de autorização (diferente do Pagador) foi associada corretamente com a transação do Antifraude Gateway.|
|400 Bad Request| Um dos campos do contrato (Tid, Nsu, Código de autorização e Data da venda) não foi informado na requisição.|
|404 Not Found|A transação do Antifraude Gateway não foi encontrada na base de dados.|
|409 Conflict|A transação da outra solução de autorização já está associada à outra transação do Antifraude Gateway.|

# Tabelas

## Tabela 1 - Provider

|Provider|Valor|
|---|---|
|Cybersource|Cybersource|
|ACI Wordwide|ReDShield|
|ClearSale|ClearSale|

## Tabela 2 - SplitingPaymentMethod

|Valor|Descrição|Provider|
|:-|:-|:-|
|None|Pagamento com um cartão apenas|ACI Wordwide|
|CardSplit|Pagamento com mais de um cartão|ACI Wordwide|
|MixedPaymentMethodSplit|Pagamento com mais de um meio de pagamento|ACI Wordwide|

## Tabela 3 - Card.Brand

|Valor|Provider|Observação|
|:-|:-|:-|
|Amex|Cybersource, ACI Wordwide|-|
|Diners|Cybersource, ACI Wordwide|-|
|Discover|Cybersource, ACI Wordwide|-|
|JCB|Cybersource, ACI Wordwide|-|
|Master|Cybersource, ACI Wordwide|-|
|Dankort|Cybersource, ACI Wordwide|-|
|Cartebleue|Cybersource, ACI Wordwide|-|
|Maestro|Cybersource, ACI Wordwide|-|
|Visa|Cybersource, ACI Wordwide|-|
|Elo|Cybersource, ACI Wordwide|-|
|Hipercard|Cybersource, ACI Wordwide|Será enviado para a Cybersource como Private Label|
|Aura|Cybersource|Será enviado para a Cybersource como Private Label|
|Naranja|Cybersource|Será enviado para a Cybersource como Private Label|
|Nevada|Cybersource|Será enviado para a Cybersource como Private Label|
|Cabal|Cybersource|Será enviado para a Cybersource como Private Label|
|Credz|Cybersource|Será enviado para a Cybersource como Private Label|
|Credsystem|Cybersource|Será enviado para a Cybersource como Private Label|
|Banese|Cybersource|Será enviado para a Cybersource como Private Label|
|Riachuelo|Cybersource|Será enviado para a Cybersource como Private Label|
|Carnet|Cybersource|Será enviado para a Cybersource como Private Label|
|Other|Cybersource|Será enviado para a Cybersource como Private Label|

## Tabela 4 - ShippingMethod

|Valor|Descrição|Provider|
|:-|:-|:-|
|SameDay|Meio de entrega no mesmo dia|Cybersource|
|SameDay|Meio de entrega no mesmo dia|ACI Worldwide, Cybersource|
|NextDay|Meio de entrega no próximo dia|ACI Worldwide, Cybersource|
|TwoDay|Meio de entrega em dois dias|ACI Worldwide, Cybersource|
|ThreeDay|Meio de entrega em três dias|ACI Worldwide, Cybersource|
|LowCost|Meio de entrega de baixo custo|ACI Worldwide, Cybersource|
|Pickup|Retirada na loja|ACI Worldwide, Cybersource|
|CarrierDesignatedByCustomer|Meio de entrega designada pelo comprador|ACI Worldwide|
|International|Meio de entrega internacional|ACI Worldwide|
|Military|Meio de entrega militar|ACI Worldwide|
|Other|Outro meio de entrega|ACI Worldwide, Cybersource|
|None|Sem meio de entrega, pois é um serviço ou assinatura|ACI Worldwide, Cybersource|

## Tabela 5 - Customer.Gender

|Valor|Descrição|Provider|
|:-|:-|:-|
|Male|Masculino|ACI Worldwide|
|Female|Feminino|ACI Worldwide|

## Tabela 6 - Customer.Status

|Valor|Descrição|Provider|
|:-|:-|:-|
|New|Identifica quando o comprador é novo na loja, nunca fez uma compra|ACI Worldwide|
|Existing|Identifica quando o comprador é existente na loja, já realizou uma compra|ACI Worldwide|

## Tabela 7 - Airline.JourneyType

|Valor|Descrição|Provider|
|:-|:-|:-|
|OneWayTrip|Viagem somente de ida|Cybersource|
|RoundTrip|Viagem de ida e volta|Cybersource|

## Tabela 8 - Airline.Passengers[n].PassengerType

|Valor|Descrição|Provider|
|:-|:-|:-|
|Adult|Adulto|ACI Worldwide, Cybersource|
|Child|Criança|ACI Worldwide, Cybersource|
|Infant|Infantil|ACI Worldwide, Cybersource|
|Youth|Adolescente|ACI Worldwide|
|Student|Estudante|ACI Worldwide|
|SeniorCitizen|Idoso|ACI Worldwide|
|Military|Militar|ACI Worldwide|

## Tabela 9 - Airline.Passengers[n].Status

|Valor|Provider|
|:-|:-|
|Standard|Cybersource|
|Gold|Cybersource|
|Platinum|Cybersource|

## Tabela 10 - CartItem[n].Risk

|Valor|Descrição|Provider|
|:-|:-|:-|
|Low|Produto associado com pouco chargebacks (default)|Cybersource|
|Normal|Produto associado com a quantidade normal de chargebacks|Cybersource|
|High|Produto associado com muito chargebacks|Cybersource|

## Tabela 11 - CartItem[n].AddressRiskVerify

|Valor|Descrição|Provider|
|:-|:-|:-|
|Yes|Em caso de divergência entre endereços de cobrança e entrega, atribui risco baixo ao pedido|Cybersource|
|No|Em caso de divergência entre endereços de cobrança e entrega, atribui risco alto ao pedido (default)|Cybersource|
|Off|Diferenças entre os endereços de cobrança e entrega não afetam a pontuação|Cybersource|

## Tabela 12 - CartItem[n].HostHedge

|Valor|Descrição|Provider|
|:-|:-|:-|
|Low|Baixa|Cybersource|
|Normal|Normal (default)|Cybersource|
|High|Alta|Cybersource|
|Off|Não irá afetar o score da análise de fraude|Cybersource|

## Tabela 13 - CartItem[n].NonSensicalHedge

|Valor|Descrição|Provider|
|:-|:-|:-|
|Low|Baixa|Cybersource|
|Normal|Normal (default)|Cybersource|
|High|Alta|Cybersource|
|Off|Não irá afetar o score da análise de fraude|Cybersource|

## Tabela 14 - CartItem[n].ObscenitiesHedge

|Valor|Descrição|Provider|
|:-|:-|:-|
|Low|Baixa|Cybersource|
|Normal|Normal (default)|Cybersource|
|High|Alta|Cybersource|
|Off|Não irá afetar o score da análise de fraude|Cybersource|

## Tabela 15 - CartItem[n].TimeHedge

|Valor|Descrição|Provider|
|:-|:-|:-|
|Low|Baixa|Cybersource|
|Normal|Normal (default)|Cybersource|
|High|Alta|Cybersource|
|Off|Não irá afetar o score da análise de fraude|Cybersource|

## Tabela 16 - CartItem[n].PhoneHedge

|Valor|Descrição|Provider|
|:-|:-|:-|
|Low|Baixa|Cybersource|
|Normal|Normal (default)|Cybersource|
|High|Alta|Cybersource|
|Off|Não irá afetar o score da análise de fraude|Cybersource|

## Tabela 17 - CartItem[n].VelocityHedge

|Valor|Descrição|Provider|
|:-|:-|:-|
|Low|Baixa|Cybersource|
|Normal|Normal (default)|Cybersource|
|High|Alta|Cybersource|
|Off|Não irá afetar o score da análise de fraude|Cybersource|

## Tabela 18 - Invoice.Tender

|Valor|Descrição|Provider|
|:-|:-|:-|
|Consumer|Cartão de crédito pessoal (default)|Cybersource|
|Corporate|Cartão de crédito corporativo|Cybersource|
|Debit|Cartão de débito|Cybersource|
|CollectDelivery|Cobrança na entrega|Cybersource|
|EletronicCheck|Cheque eletrônico|Cybersource|
|PaymentP2P|Pagamento de pessoa para pessoa|Cybersource|
|PrivateLabel|Pagamento com cartão de crédito privado|Cybersource|
|Other|Pagamentos com outros métodos|Cybersource|

## Tabela 19 - Status

|Valor|Descrição|Provider|
|:-|:-|:-|
|Accept|Transação aceita após análise de fraude|ACI Worldwide, Cybersource|
|Review|Transação em revisão após análise de fraude|ACI Worldwide, Cybersource|
|Reject|Transação rejeitada após análise de fraude|ACI Worldwide, Cybersource|
|Pendent|Transação pendente, pois ao enviar a mesma para análise de fraude ocorreu um timeout na resposta entre Braspag e Cybersource|Cybersource|
|Unfinished|Transação não finalizada por algum motivo de validação de contrato ou erro interno <br/> Se na resposta de uma transação analisada na Cybersource o campo `ProviderAnalysisResult.ProviderStatus` for igual a **REJECT** e o campo `ProviderAnalysisResult.ProviderCode` diferente de **481**, o status da transação será **Unfinished**|ACI Worldwide, Cybersource|
|ProviderError|Transação com erro no provedor ao ser enviada para análise|ACI Worldwide, Cybersource|

## Tabela 20 - ProviderStatus

|Valor|Descrição|Provider|De-Para com o campo `Status` (Status da transação no Antifraude Gateway Braspag)|
|:-|:-|:-|:-|
|APPROVE|Transação aprovada no provedor|ACI Worldwide|Accept|
|ACCEPT|Transação aceita no provedor|ACI Worldwide, Cybersource|Accept|
|PEND|Transação em revisão no provedor|ACI Worldwide|Review|
|CHALLENGE|Transação em revisão no provedor|ACI Worldwide|Review|
|REVIEW|Transação em revisão no provedor|Cybersource|Review|
|CANCEL|Transação rejeitada no provedor|ACI Worldwide|Reject|
|DENY|Transação rejeitada no provedor|ACI Worldwide|Reject|
|REJECT|Transação rejeitada no provedor|Cybesource|Reject|
|ENETLP|Transação com erro no provedor|ACI Worldwide|ProviderError|
|ENORSP|Transação com erro no provedor|ACI Worldwide|ProviderError|
|ERROR|Transação com erro no provedor|ACI Worldwide, Cybersource|ProviderError|

## Tabela 21 - ProviderAnalysisResult.ProviderCode

|Valor|Descrição|Provider|
|:-|:-|:-|
|100|Operação realizada com sucesso|Cybersource|
|101|A transação enviada para análise de fraude está faltando um ou mais campos necessários <br/> Verificar no response o campo `ProviderAnalysisResult.Missing` <br/> Possível ação: Reenviar a transação com a informação completa|Cybersource|
|102|A transação enviada para análise de fraude possui um ou mais campos com valores inválidos <br/> Verificar no response o campo `ProviderAnalysisResult.Invalid` <br/> Possível ação: Reenviar a transação com as informações corretas|Cybersource|
|150|Erro interno <br/> Possível ação: Aguarde alguns minutos e tente reenviar a transação|Cybersource|
|151|A transação foi recebida, mas ocorreu time-out no servidor. Este erro não inclui time-out entre o cliente e o servidor <br/> Possível ação: Aguarde alguns minutos e tente reenviar a transação|Cybersource|
|152|O pedido foi recebido, mas ocorreu time-out <br/> Possível ação: Aguarde alguns minutos e tente reenviar a transação|Cybersource|
|202|Transação recusada pois o cartão expirou ou a data de validade não coincide com a correta <br/> Possível ação: Solicite outro cartão ou outra forma de pagamento|Cybersource|
|231|Transação recusada pois o cartão é inválido <br/> Possível ação: Solicite outro cartão ou outra forma de pagamento|Cybersource|
|234|Problema com a configuração da loja na Cybersource <br/> Possível ação: Entre em contato com o suporte para corrigir o problema de configuração|Cybersource|
|400|A pontuação de fraude ultrapassa o seu limite <br/> Possível ação: Reveja a transação do comprador|Cybersource|
|480|A transação foi marcada como revisão pelo DM (Decision Manager)|Cybersource|
|481|A transação foi rejeitada pelo DM (Decision Manager)|Cybersource|

## Tabela 22 - ProviderAnalysisResult.AfsReply.AddressInfoCode

|Valor|Descrição|Provider|
|:-|:-|:-|
|COR-BA|O endereço de cobrança pode ser normalizado|Cybersource|
|COR-SA|O endereço de entrega pode ser normalizado|Cybersource|
|INTL-BA|O país do endereço de cobrança está fora dos EUA|Cybersource|
|INTL-SA|O país do endereço de entrega está fora dos EUA|Cybersource|
|MIL-USA|Endereço militar nos EUA|Cybersource|
|MM-A|Os endereços de cobrança e entrega usam nomes de ruas diferentes|Cybersource|
|MM-BIN|O BIN do cartão (os seis primeiros dígitos do número do cartão) não corresponde ao país|Cybersource|
|MM-C|Os endereços de cobrança e entrega usam cidades diferentes|Cybersource|
|MM-CO|Os endereços de cobrança e entrega usam países diferentes|Cybersource|
|MM-ST|Os endereços de cobrança e entrega usam estados diferentes|Cybersource|
|MM-Z|Os endereços de cobrança e entrega usam códidos postais diferentes|Cybersource|
|UNV-ADDR|O endereço é inverificável|Cybersource|

## Tabela 23 - ProviderAnalysisResult.AfsReply.AfsFactorCode

|Valor|Descrição|Provider|
|:-|:-|:-|
|A|Mudança de endereço excessiva. O comprador mudou o endereço de cobrança duas ou mais vezes nos últimos seis meses|Cybersource|
|B|BIN do cartão ou autorização de risco. Os fatores de risco estão relacionados com BIN de cartão de crédito e/ou verificações de autorização do cartão|Cybersource|
|C|Elevado números de cartões de créditos. O comprador tem usado mais de seis números de cartões de créditos nos últimos seis meses|Cybersource|
|D|Impacto do endereço de e-mail. O comprador usa um provedor de e-mail gratuito ou o endereço de email é arriscado|Cybersource|
|E|Lista positiva. O comprador está na sua lista positiva|Cybersource|
|F|Lista negativa. O número da conta, endereço, endereço de e-mail ou endereço IP para este fim aparece sua lista negativa|Cybersource|
|G|Inconsistências de geolocalização. O domínio do comprador de e-mail, número de telefone, endereço de cobrança, endereço de envio ou endereço IP é suspeito|Cybersource|
|H|Excessivas mudanças de nome. O comprador mudou o nome duas ou mais vezes nos últimos seis meses|Cybersource|
|I|Inconsistências de internet. O endereço IP e de domínio de e-mail não são consistentes com o endereço de cobrança|Cybersource|
|N|Entrada sem sentido. O nome do comprador e os campos de endereço contém palavras sem sentido ou idioma|Cybersource|
|O|Obscenidades. Dados do comprador contém palavras obscenas|Cybersource|
|P|Identidade morphing. Vários valores de um elemento de identidade estão ligados a um valor de um elemento de identidade diferentes. Por exemplo, vários números de telefones estão ligados a um número de conta única|Cybersource|
|Q|Inconsistências do telefone. O número de telefone do comprador é suspeito|Cybersource|
|R|Ordem arriscada. A transação, o comprador e o lojista mostram informações correlacionadas de alto risco|Cybersource|
|T|Cobertura Time. O comprador está a tentar uma compra fora do horário esperado|Cybersource|
|U|Endereço não verificável. O endereço de cobrança ou de entrega não pode ser verificado|Cybersource|
|V|O cartão foi usado muitas vezes nos últimos 15 minutos|Cybersource|
|W|Marcado como suspeito. O endereço de cobrança ou de entrega é semelhante a um endereço previamente marcado como suspeito|Cybersource|
|Y|O endereço, cidade, estado ou país dos endereços de cobrança e entrega não se correlacionam|Cybersource|
|Z|Valor inválido. Como a solicitação contém um valor inesperado, um valor padrão foi substituído. Embora a transação ainda possa ser processada, examinar o pedido com cuidado para detectar anomalias|Cybersource|

## Tabela 24 - ProviderAnalysisResult.AfsReply.CardAccountType

|Valor|Descrição|Provider|
|:-|:-|:-|
|CN|Cartão privado|Cybersource|
|CP|Cartão corporativo|Cybersource|

## Tabela 25 - ProviderAnalysisResult.AfsReply.HotListInfoCode

|Valor|Descrição|Provider|
|:-|:-|:-|
|CON-POSNEG|A ordem disparada bate tanto com a lista positiva e negativa. O resultado da lista positiva sobrescreve a lista negativa|Cybersource|
|NEG-BA|O endereço de cobrança está na lista negativa|Cybersource|
|NEG-BCO|O país do endereço de cobrança está na lista negativa|Cybersource|
|NEG-BIN|O BIN do cartão de crédito (os seis primeiros dígitos do número do cartão) está na lista negativa|Cybersource|
|NEG-BINCO|O país em que o cartão de crédito foi emitido está na lista negativa|Cybersource|
|NEG-BZC|O código postal do endereço de cobrança está na lista negativa|Cybersource|
|NEG-CC|O número de cartão de crédito está na lista negativa|Cybersource|
|NEG-EM|O endereço de e-mail está na lista negativa|Cybersource|
|NEG-EMCO|O país em que o endereço de e-mail está localizado na lista negativa|Cybersource|
|NEG-EMDOM|O domínio de e-mail (por exemplo, mail.example.com) está na lista negativa|Cybersource|
|NEG-FP|O device fingerprint está na lista negativa|Cybersource|
|NEG-HIST|A transação foi encontrada na lista negativa|Cybersource|
|NEG-ID|Identificador do comprador (CPF ou CNPJ) está na lista negativa|Cybersource|
|NEG-IP|O endereço IP (por exemplo, 10.1.27.63) está na lista negativa|Cybersource|
|NEG-IP3|O endereço IP de rede (por exemplo, 10.1.27.63) está na lista negativa. Um endereço de IP da rede inclui até 256 endereços IP|Cybersource|
|NEG-IPCO|O país em que o endereço IP está localizado está na lista negativa|Cybersource|
|NEG-PEM|O endereço de e-mail do passageiro está na lista negativa|Cybersource|
|NEG-PH|O número do telefone está na lista negativa|Cybersource|
|NEG-PID|Identificador do passageiro está na lista negativa|Cybersource|
|NEG-PPH|O número do telefone do passageiro está na lista negativa|Cybersource|
|NEG-SA|O endereço de entrega está na lista negativa|Cybersource|
|NEG-SCO|O país do endereço de entrega está na lista negativa|Cybersource|
|NEG-SZC|O código postal do endereço de entrega está na lista negativa|Cybersource|
|POS-TEMP|O comprador está temporariamente na lista positiva|Cybersource|
|POS-PERM|O comprador está permanente na lista positiva|Cybersource|
|REV-BA|O endereço de cobrança esta na lista de revisão|Cybersource|
|REV-BCO|O país do endereço de cobrança está na lista de revisão|Cybersource|
|REV-BIN|O BIN do cartão de crédito (os seis primeiros dígitos do número do cartão) está na lista de revisão|Cybersource|
|REV-BINCO|O país em que o cartão de crédito foi emitido está na lista de revisão|Cybersource|
|REV-BZC|O código postal do endereço de cobrança está na lista de revisão|Cybersource|
|REV-CC|O número do cartão de crédito está na lista de revisão|Cybersource|
|REV-EM|O endereço de e-mail está na lista de revisão|Cybersource|
|REV-EMCO|O país em que o endereço de e-mail está localizado está na lista de revisão|Cybersource|
|REV-EMDOM|O domínio de e-mail (por exemplo, mail.example.com) está na lista de revisão|Cybersource|
|REV-FP|O device fingerprint está na lista de revisão|Cybersource|
|REV-ID|Identificador do comprador (CPF ou CNPJ) está na lista de revisão|Cybersource|
|REV-IP|O endereço IP (por exemplo, 10.1.27.63) está na lista de revisão|Cybersource|
|REV-IP3|O endereço IP de rede (por exemplo, 10.1.27.63) está na lista de revisão. Um endereço de IP da rede inclui até 256 endereços IP|Cybersource|
|REV-IPCO|O país em que o endereço IP está localizado está na lista de revisão|Cybersource|
|REV-PEM|O endereço de e-mail do passageiro está na lista de revisão|Cybersource|
|REV-PH|O número do telefone está na lista de revisão|Cybersource|
|REV-PID|Identificador do passageiro está na lista de revisão|Cybersource|
|REV-PPH|O número do telefone do passageiro está na lista de revisão|Cybersource|
|REV-SA|O endereço de entrega está na lista de revisão|Cybersource|
|REV-SCO|O país do endereço de entrega está na lista de revisão|Cybersource|
|REV-SZC|O código postal do endereço de entrega está na lista de revisão|Cybersource|

## Tabela 26 - ProviderAnalysisResult.AfsReply.IdentityInfoCode

|Valor|Descrição|Provider|
|:-|:-|:-|
|MORPH-B|O mesmo endereço de cobrança tem sido utilizado várias vezes com identidades de múltiplos compradores|Cybersource|
|MORPH-C|O mesmo cartão de crédito tem sido utilizado várias vezes com identidades de múltiplos compradores|Cybersource|
|MORPH-E|O mesmo endereço de e-mail tem sido utilizado várias vezes com identidades de múltiplos compradores|Cybersource|
|MORPH-I|O mesmo endereço IP tem sido utilizado várias vezes com identidades de múltiplos compradores|Cybersource|
|MORPH-P|O mesmo número de telefone tem sido usado várias vezes com identidades de múltiplos compradores|Cybersource|
|MORPH-S|O mesmo endereço de entrega tem sido utilizado várias vezes com identidades de múltiplos compradores|Cybersource|

## Tabela 27 - ProviderAnalysisResult.AfsReply.InternetInfoCode

|Valor|Descrição|Provider|
|:-|:-|:-|
|FREE-EM|O endereço de e-mail do comprador é de um provedor de e-mail gratuito|Cybersource|
|INTL-IPCO|O país do endereço de e-mail do comprador está fora dos EUA|Cybersource|
|INV-EM|O endereço de e-mail do comprador é inválido|Cybersource|
|MM-EMBCO|O domínio do endereço de e-mail do comprador não é consistente com o país do endereço de cobrança|Cybersource|
|MM-IPBC|O endereço de e-mail do comprador não é consistente com a cidade do endereço de cobrança|Cybersource|
|MM-IPBCO|O endereço de e-mail do comprador não é consistente com o país do endereço de cobrança|Cybersource|
|MM-IPBST|O endereço IP do comprador não é consistente com o estado no endereço de cobrança. No entanto, este código de informação não pode ser devolvido quando a inconsistência é entre estados imediatamente  adjacentes|Cybersource|
|MM-IPEM|O endereço de e-mail do comprador não é consistente com o endereço IP|Cybersource|
|RISK-EM|O domínio do e-mail do comprador (por exemplo, mail.example.com) está associado com alto risco|Cybersource|
|UNV-NID|O endereço IP do comprador é de um proxy anônimo. Estas entidades escondem completamente informações sobre o endereço de IP|Cybersource|
|UNV-RISK|O endereço IP é de origem de risco|Cybersource|
|UNV-EMBCO|O país do endereço de e-mail não corresponde ao país do endereço de cobrança|Cybersource|

## Tabela 28 - ProviderAnalysisResult.AfsReply.PhoneInfoCode

|Valor|Descrição|Provider|
|:-|:-|:-|
|MM-ACBST|O número de telefone do comprador não é consistente com o estado no endereço de cobrança|Cybersource|
|RISK-AC|O código de área do comprador está associado com risco alto|Cybersource|
|RISK-PH|O número de telefone dos EUA. ou do Canadá é incompleto, ou uma ou mais partes do número são arriscadas|Cybersource|
|TF-AC|O número do telefone utiliza um código de área toll-free (grátis)|Cybersource|
|UNV-AC|O código de área é inválido|Cybersource|
|UNV-OC|O código de área e/ou o prefixo de telefone são/é inválido|Cybersource|
|UNV-PH|O número do telefone é inválido|Cybersource|

## Tabela 29 - ProviderAnalysisResult.AfsReply.SuspiciousInfoCode

|Valor|Descrição|Provider|
|:-|:-|:-|
|BAD-FP|O dispositivo é arriscado|Cybersource|
|INTL-BIN|O cartão de crédito foi emitido fora dos EUA|Cybersource|
|MM-TZTLO|Fuso horário do dispositivo é incompatível com os fusos horários do país|Cybersource|
|MUL-EM|O comprador tem usado mais de quatro endereços de e-mail diferentes|Cybersource|
|NON-BC|A cidade do endereço de cobrança é sem sentido|Cybersource|
|NON-FN|O primeiro nome do comprador é sem sentido|Cybersource|
|NON-LN|O último nome do comprador é sem sentido|Cybersource|
|OBS-BC|A cidade do endereço de cobrança contem obscenidades|Cybersource|
|OBS-EM|O endereço de e-mail contem obscenidades|Cybersource|
|RISK-AVS|O resultado do combinado do teste AVS e endereço de cobrança normalizado são arriscados, o resultado AVS indica uma correspondência exata, mas o endereço de cobrança não é entregável|Cybersource|
|RISK-BC|A cidade do endereço de cobrança possui caracteres repetidos|Cybersource|
|RISK-BIN|No passado, este BIN do cartão de crédito (os seis primeiros dígitos do número do cartão) mostrou uma elevada incidência de fraude|Cybersource|
|RISK-DEV|Algumas das características do dispositivo são arriscadas|Cybersource|
|RISK-FN|Nome e último nome do comprador contêm combinações de letras improváveis|Cybersource|
|RISK-LN|Nome do meio ou último nome do comprador contêm combinações de letras improváveis|Cybersource|
|RISK-PIP|O endereço IP do proxy é arriscado|Cybersource|
|RISK-SD|A inconsistência nos países dos endereços cobrança e entrega é arriscada|Cybersource|
|RISK-TB|O dia e a hora da ordem associada ao endereço de cobrança é arriscado|Cybersource|
|RISK-TIP|O verdadeiro endereço IP é arriscado|Cybersource|
|RISK-TS|O dia e a hora da ordem associada ao endereço de entrega é arriscado|Cybersource|

## Tabela 30 - ProviderAnalysisResult.AfsReply.VelocityInfoCode

|Valor|Descrição|Provider|
|:-|:-|:-|
|VEL-ADDR|Diferente estados dos endereços de cobrança e/ou entrega (EUA e Canadá apenas) têm sido usadas várias vezes com o número do cartão de crédito e/ou endereço de email|Cybersource|
|VEL-CC|Diferentes números de cartões de créditos foram usados várias vezes com o mesmo nome ou endereço de email|Cybersource|
|VEL-NAME|Diferentes nomes foram usados várias vezes com o mesmo número do cartão de crédito e/ou endereço de email|Cybersource|
|VELS-CC|O número do cartão de crédito tem sido utilizado várias vezes durante um intervalo curto|Cybersource|
|VELI-CC|O número do cartão de crédito tem sido utilizado várias vezes durante um intervalo médio|Cybersource|
|VELL-CC|O número do cartão de crédito tem sido utilizado várias vezes durante um intervalo longo|Cybersource|
|VELV-CC|O número do cartão de crédito tem sido utilizado várias vezes durante um intervalo muito longo|Cybersource|
|VELS-EM|O endereço de e-mail tem sido utilizado várias vezes durante um intervalo curto|Cybersource|
|VELI-EM|O endereço de e-mail tem sido utilizado várias vezes durante um intervalo médio|Cybersource|
|VELL-EM|O endereço de e-mail tem sido utilizado várias vezes durante um intervalo longo|Cybersource|
|VELV-EM|O endereço de e-mail tem sido utilizado várias vezes durante um intervalo muito longo|Cybersource|
|VELS-FP|O device fingerprint tem sido utilizado várias vezes durante um intervalo curto|Cybersource|
|VELI-FP|O device fingerprint tem sido utilizado várias vezes durante um intervalo médio|Cybersource|
|VELL-FP|O device fingerprint tem sido utilizado várias vezes durante um intervalo longo|Cybersource|
|VELV-FP|O device fingerprint tem sido utilizado várias vezes durante um intervalo muito longo|Cybersource|
|VELS-IP|O endereço IP tem sido utilizado várias vezes durante um intervalo curto|Cybersource|
|VELI-IP|O endereço IP tem sido utilizado várias vezes durante um intervalo médio|Cybersource|
|VELL-IP|O endereço IP tem sido utilizado várias vezes durante um intervalo longo|Cybersource|
|VELV-IP|O endereço IP tem sido utilizado várias vezes durante um intervalo muito longo|Cybersource|
|VELS-SA|O endereço de entrega tem sido utilizado várias vezes durante um intervalo curto|Cybersource|
|VELI-SA|O endereço de entrega tem sido utilizado várias vezes durante um intervalo médio|Cybersource|
|VELL-SA|O endereço de entrega tem sido utilizado várias vezes durante um intervalo longo|Cybersource|
|VELV-SA|O endereço de entrega tem sido utilizado várias vezes durante um intervalo muito longo|Cybersource|
|VELS-TIP|O endereço IP verdadeiro tem sido utilizado várias vezes durante um intervalo curto|Cybersource|
|VELI-TIP|O endereço IP verdadeiro tem sido utilizado várias vezes durante um intervalo médio|Cybersource|
|VELL-TIP|O endereço IP verdadeiro tem sido utilizado várias vezes durante um intervalo longo|Cybersource|

## Tabela 31 - ProviderAnalysisResult.AfsReply.IpRoutingMethod

|Valor|Descrição|Provider|
|:-|:-|:-|
|Anonymizer|Endereços de IP estão escondidos porque o comprador é extremamente cauteloso, quer privacidade absoluta ou é fraudulento|Cybersource|
|AOL, AOL dialup, AOL POP and AOL proxy|Membros da AOL. Na maioria dos casos, o país pode ser identificado, mas o estado e cidade não podem|Cybersource|
|Cache proxy|Proxy usado através de um acelerador da Internet ou de uma distribuição de conteúdo de serviço. O comprador pode estar localizado em um país diferente do indicado pelo endereço de IP|Cybersource|
|Fixed|O endereço de IP está próximo ou no mesmo local que o comprador|Cybersource|
|International proxy|Proxy que contém tráfego de vários países. O comprador pode estar localizado em um país diferente do indicado pelo endereço de IP. Em muitos casos, redes corporativas estão roteando o tráfego de escritórios internacionais através de um ponto central, muitas vezes a sede corporativa|Cybersource|
|Mobile gateway|Gateway para conectar dispositivos móveis à internet. Muitas operadoras, especialmente na Europa, atendem mais do que um país e tráfego ocorre através de hubs de rede centralizados. O comprador pode estar localizado em um país diferente do indicado pelo endereço de IP|Cybersource|
|POP|Discagem do comprador em um ISP regional provavelmente perto da localização do endereço de IP, mas possivelmente através de limites geográficos|Cybersource|
|Regional proxy|Proxy que contém tráfego de vários estados dentro de um único país. O comprador pode estar localizado em um estado diferente do indicado pelo endereço de IP. Em muitos casos, redes corporativas estão roteando o tráfego de escritórios internacionais através de um ponto central, muitas vezes a sede corporativa|Cybersource|
|Satellite|Conexões por satélite. Se o uplink e o downlink estiverem cadastrados, o método roteamento é considerado padrão porque o remetente é conhecido. No entanto, se o downlink não está registrado, o comprador pode estar em qualquer lugar dentro do feixe padrão do satélite, que pode abranger um continente ou mais|Cybersource|
|SuperPOP|O comprador está discando em um ISP multi-estatal ou multinacional que provavelmente não é provável a localização do endereço de IP. O comprador pode estar discando através de limites geográficos|Cybersource|
|No value returned|O tipo de roteamento é desconhecido|Cybersource|

## Tabela 32 - ProviderAnalysisResult.DecisionReply.ActiveProfileReply.RulesTriggered[n].Decision

|Valor|Provider|
|:-|:-|
|ACCEPT|Cybersource|
|REVIEW|Cybersource|
|REJECT|Cybersource|
|ERROR|Cybersource|

## Tabela 33 - ProviderAnalysisResult.DecisionReply.ActiveProfileReply.RulesTriggered[n].Evaluation

|Valor|Descrição|Provider|
|:-|:-|:-|
|T|A regra é verdadeira|Cybersource|
|F|A regra é falsa|Cybersource|
|N|A regra não pode ser avaliada porque os dados são insuficientes|Cybersource|
|E|A regra não pode ser avaliada porque ocorreu um erro|Cybersource|

## Tabela 34 - MerchantDefinedData (Cybersource)

> Nível de Relevância <br/> 1 - Relevante <br/> 2 - Muito Relevante <br/> 3 - Extremamente Relevante <br/><br/>
> Conforme nível de relevância dos campos e possibilidade de desenho da estratégia de risco de acordo com a necessidade do seu negócio, na validação das transações de testes os mesmos serão cobrados caso não sejam enviaos. Com isso, solicitamos uma análise prévia da documentação e sinalização dos campos que não serão possíveis de serem enviados.<br/><br/>
> No caso de não possuir o dado para enviar, pedimos a gentileza de não enviar o campo correspondente como vazio, ou seja, apenas não enviar.

|Key|Value|Tipo|Nível de Relevância|Segmento|
|:-|:-|:-|:-:|:-|
|1|Cliente efetuou Login <br/> Se o cliente final logou no site para comprar, enviar: o login dele <br/> Se fez compra como visitante, enviar: Guest <br/> Se a venda foi feita direto por um terceiro, um agente por exemplo, não enviar o campo|string|2|Todos|
|2|Quantidade em dias que o cliente é seu cliente <br/> Ex.: 314|int|3|Todos|
|3|Quantidade de parcelas do pedido|int|3|Todos|
|4|Canal de Venda <br/> Possíveis valores: <br/> Call Center -> compra pelo telefone <br/> Web -> compra pela web <br/> Portal -> um agente fazendo a compra para o cliente <br/> Quiosque -> compras em quiosques <br/> Movel -> compras feitas em celulares ou tablets|string|3|Todos|
|5|Enviar o código do cupom/desconto caso o cliente utilize na compra|string|1|Todos|
|6|Quantidade em dias desde a última compra realizada pelo cliente <br/> Ex.: 55|int|3|Todos|
|7|Código ou nome do seller (vendedor)|string|1|Todos|
|8|Tentativas realizadas pelo cliente de efetuar o pagamento do mesmo pedido, podendo ser com diferentes cartões de créditos e/ou através de outros meios de pagamentos|int|2|Todos|
|9|Identifica se cliente irá retirar o produto na loja <br/> Possíveis valores: SIM ou NAO|string|3|Varejo ou Cosméticos|
|10|Identifica se o pagamento será realizado por outra pessoa que não esteja presente na viagem ou pacote <br/> Possíveis valores: SIM ou NAO|string|3|Aéreo ou Turismo|
|11|Categoria do hotel (quantas estrelas) <br/> Possíveis valores: <br/> 1 -> Simples <br/> 2 -> Econômico <br> 3 -> Turismo <br/> 4 -> Superior <br/> 5 -> Luxo|int|3|Turismo|
|12|Quantidade em dias desde a data da compra até a data do checkin no hotel <br/> Ex.: 123|int|3|Turismo|
|13|Quantidade de diárias no hotel <br/> Ex.: 5|int|3|Turismo|
|14|Categoria da viagem ou pacote <br> Possíveis valores: Nacional ou Internacional ou Nacional/Internacional|string|3|Aéreo ou Turismo|
|15|Nome da companhia áerea / locadora de carro / hotel <br/> Enviar o nome de cada uma das empresas, separado por /|string|2|Aéreo ou Turismo|
|16|Código PNR da reserva <br/> Quando houver uma alteração da reserva para este PNR, com antecipação da data de voo, é importante fazer uma nova análise de fraude enviando este PNR novamente|string|3|Aérea|
|17|Identifica se houve antecipação de reserva <br/> Possíveis valores: SIM ou NAO <br/> Se sim, fundamental o envio também do campo 16 - Código PNR da reserva|string|3|Aéreo|
|18|Categoria do veículo alugado <br/> Possíveis valores: <br/> 1 - Básico <br/> 2 - Esportivo <br/> 3 - Prime <br/> 4 - Utilitário <br/> 5 - Blindado|string|3|Turismo|
|19|Identifica se o pacote se à cruzeiro <br/> Possíveis valores: SIM ou NAO|string|2|Turismo|
|20|Decisão da análise de fraude referente a última compra <br/> Possíveis valores: ACEITA ou REJEITADA|string|3|Todos|
|21|Valor do frete <br/> Ex.: 10599 = r$ 105,99|long|1|Varejo ou Cosméticos|
|22|Código da loja onde o produto será retirado <br/> Este campo deverá ser enviado quando o campo 9 for enviado igual a SIM|string|3|Varejo ou Cosméticos|
|23|Sufixo (4 últimos dígitos) do cartão de crédito|int|1|Todos|
|24|Quantidade de dias desde a primeira compra realizada pelo cliente <br/> Ex.: 150|int|3|Todos|
|25|Sexo do cliente <br/> Possíveis valores: <br/> F -> Feminino <br/> M -> Masculino|string|2|Todos|
|26|Bin (6 primeiros dígitos) do cartão de crédito|int|1|Todos|
|27|Tipo do logradouro do endereço de entrega <br/> Possíveis valores: <br/> R -> Residencial <br/> C -> Comercial|string|2|Todos|
|28|Tempo médio em minutos que o cliente levou para realizar a compra|int|2|Todos|
|29|Quantidade de tentativas que o cliente realizou para efetuar login|int|2|Todos|
|30|Quantidade de páginas web que o cliente visitou anteriormente a compra referente a 30 minutos passados|int|2|Todos|
|31|Quantidade de trocas de números de cartão de crédito que o cliente efetuou para realizar o pagamento do pedido|int|2|Todos|
|32|Identifica se o e-mail foi colado ou digitado <br/> Possíveis valores: Digitado ou Colado|string|3|Todos|
|33|Identifica se o número do cartão de crédito foi colado ou digitado <br/> Possíveis valores: Digitado ou Colado|string|3|Todos|
|34|Identifica se o e-mail foi confirmado para ativação de conta <br/> Possíveis valores: SIM ou NAO|string|2|Todos|
|35|Identifica o tipo de cliente <br/> Possíveis valores: Local ou Turista|string|2|Turismo|
|36|Identifica se foi utilizado cartão presente (GiftCard) na compra como forma de pagamento <br/> Possíveis valores: SIM ou NAO|string|1|Todos|
|37|Meio de envio do pedido <br/> Possíveis valores: Sedex <br/> Sedex 10 <br/> 1 Dia <br/> 2 Dias <br/> Motoboy <br/> Mesmo Dia <br/>|string|3|Varejo ou Cosméticos|
|38|Número do telefone do cliente identificado através da bina quando venda realizada através do canal de venda igual a Call Center <br/> Formato: DDIDDDNúmero - Ex.: 552121114720|string|3|Todos|
|39|Nome de usuário do Call Center <br/> Este campo deverá ser enviado quando campo 4 for enviado igual a Call Center|string|1|Todos|
|40|Comentários inseridos quando pedido for presente|string|1|Todos|
|41|Tipo do documento <br/> Possíveis valores: CPF ou CNPJ ou Passaporte|string|2|Todos|
|42|Idade do cliente|int|2|Todos|
|43|Faixa de rendimento do cliente <br/> Ex.: 100000 = r$ 1.000,00|long|2|Todos|
|44|Quantidade histórica de compras realizadas pelo cliente|int|3|Todos| 
|45|Identifica se é uma compra realizada por funcionário <br/> Possíveis valores: SIM ou NAO|string|2|Todos|
|46|Nome impresso (portador) no cartão de crédito|string|3|Todos|
|47|Identifica se o cartão é private label <br/> Possíveis valores: SIM ou NAO|string|2|Todos|
|48|Quantidade de meios de pagamentos utilizados para efetuar a compra|int|2|Todos|
|49|Média das compras realizadas nos últimos 6 meses <br/> Ex.: 159050 = r$ 1.590,99|long|3|Todos|
|50|Fator de desvio de valor da compra atual sobre a média dos últimos 6 meses <br/> Exemplo de formato: 2.5 (sem separador de milhares, separar decimal por ponto)|float|3|Todos|
|51|Identifica se é um cliente VIP com tratamento de risco diferenciado ou lista positiva <br/> Possíveis valores: SIM ou NAO|string|3|Todos|
|52|Categoria do produto <br/> Possíveis valores: <br/> Animais e Bichos de Estimação <br/> Roupas e Acessórios <br/> Negócios e Indústria <br/> Câmeras e Óticas <br/> Eletrônicos <br/> Comidas, Bebidas e Cigarro <br/> Móveis <br/> Ferramentas <br/> Saúde e Beleza <br/> Casa e Jardim <br/> Malas e Bagagens <br/> Adulto <br/> Armas e Munição <br/> Materiais de Escritório <br/> Religião e Cerimoniais <br/> Software <br/> Equipamentos de Esporte <br/> Brinquedos e Jogos <br/> Veículos e Peças <br/> Livros <br/> DVDs e Vídeos <br/> Revistas e Jornais <br/> Música <br/> Outras Categorias Não Especificadas|string|2|Todos|
|53|Identifica se existe rotina de confirmação de celular por SMS <br/> Possíveis valores: SIM ou NAO|string|2|Todos|
|54|Qual a 2ª forma de pagamento|string|2|Todos|
|55|Qual a 3ª forma de pagamento|string|2|Todos|
|56|Se 2ª forma de pagamento for cartão de crédito, enviar a bandeira|string|1|Todos|
|57|Se 3ª forma de pagamento for cartão de crédito, enviar a bandeira|string|1|Todos|
|58|Se 2ª forma de pagamento, informar o valor pago <br/> Ex.: 128599 = r$ 1.285,99|long|2|Todos|
|59|Se 3ª forma de pagamento, informar o valor pago <br/> Ex.: 59089 = r$ 590,89|long|2|Todos|
|60|Quantidade em dias desde a data da última alteração <br/> Ex.: 57|int|3|Todos|
|61|Identifica se houve alteração cadastral|string|1|Todos|
|62|Quantidade de pontos trocados na última compra|long|3|Fidelidade|
|63|Quantidade de pontos restantes no saldo|long|2|Fidelidade|
|64|Quantidade de dias desde a última troca de pontos|long|2|Fidelidade|
|65|Identificador do cliente no programa de fidelidade|string|2|Fidelidade|
|66|Quantidade de minutos recarregados nos últimos 30 dias|long|2|Digital Goods|
|67|Quantidade de recargas realizadas nos últimos 30 dias|long|2|Digital Goods|
|68|Quantidade em dias entre a data de partida e a data de retorno|int|2|Aéreo|
|69|Quantidade de passageiros viajando independente da faixa etária|int|2|Aéreo|
|70|Identificador do voô|string|1|Aéreo|
|71|Número de infants viajando|int|2|Aéreo|
|72|Número de crianças viajando|int|2|Aéreo|
|73|Número de adultos viajando|int|2|Aéreo|
|74|Identifica se é um passageiro frequente (Frequently Flyer) <br/> Possíveis valores: SIM ou NAO|string|2|Aéreo|
|75|Identificar do passageiro frequente (Frequently Flyer Number)|string|2|Aéreo|
|76|Categoria do passageiro frequente (Frequently Flyer) <br/> Esta categoria pode variar de acordo com a companhia aérea|int|2|Aéreo|
|77|Dia da semana do embarque <br/> Possíveis valores: Sunday (Domingo) <br/> Monday (Segunda-feira) <br/> Tuesday (Terça-feira) <br/> Wednesday (Quarta-feira) <br/> Thursday (Quinta-feira) <br/> Friday (Sexta-feira) <br/> Saturday (Sábado)|string|2|Aéreo|
|78|Código da companhia aérea <br/> Ex.: JJ ou LA ou AA ou UA ou G3 e etc|string|1|Aéreo|
|79|Classe tarifária da passagem <br/> Ex.: W ou Y ou N e etc|string|2|Aéreo|
|80|Número do celular do passageiro <br/> Ex.: Formato: DDIDDDNúmero - Ex.: 5521976781114|string|2|Aéreo|
|81|Identifica se o dono do cartão de crédito irá viajar <br/> Possíveis valores: SIM ou NAO|string|3|Aéreo|
|82|Identifica se o seller (vendedor) irá trabalhar com revisão manual ou não <br/> Possíveis valores: SIM ou NAO|string|1|Todos|
|83|Segmento de negócio <br/> Ex.: Varejo|string|2|Todos|
|84|Nome da plataforma integrada a API Antifraude Gateway Braspag <br/> Caso seja uma integração direta entre a loja e Braspag, enviar valor igual a PROPRIA|string|3|Todos|
|85 a 89|Campos livres e definidos junto ao provedor de antifraude, conforme as regras de negócio|-|-|-|
|90 a 100|Reservados|-|-|-|

## Tabela 35 - MerchantDefinedData (ACI Worldwide)

|Key|Value|Tipo|Tamanho|
|:-|:-|:-|:-|
|1 a 3|Reservado|-|-|
|4 a 8|Campos livres e definidos junto ao provedor de antifraude, conforme as regras de negócio|var|256|
|9 a 13|Reservado|-|-|
|14|Segment|MCC (Merchant Category Code) da sua loja|int|-|
|15 a 20|Campos livres e definidos junto ao provedor de antifraude, conforme as regras de negócio|var|30|
|21|Reservado|-|-|
|22|Campo livre e definido junto ao provedor de antifraude, conforme as regras de negócio|var|30|
|23|Reservado|-|-|
|24|Campo livre e definido junto ao provedor de antifraude, conforme a regras de negócio|var|30|
|25|Reservado|-|-|

## Tabela 36 - CartItem[n].Category

|Valor|Descrição|Provider|
|:-|:-|:-|
|AdultContent|Conteúdo adulto|Cybersource|
|Coupon|Cupom aplicado para todo o pedido|Cybersource|
|Default|Valor default para o tipo do produto. Quando não enviado nenhum outro valor, assume-se o a categoria senda esta|Cybersource|
|EletronicGood|Produto eletrônico diferente de software|Cybersource|
|EletronicSoftware|Softwares distribuídos eletronicamente via download|Cybersource|
|GiftCertificate|Vale presente|Cybersource|
|HandlingOnly|Taxa que você cobra do seu cliente para cobrir os seus custos administrativos de venda. Ex.: Taxa de conveniência / Taxa de instalação|Cybersource|
|Service|Serviço que será realizado para o cliente|Cybersource|
|ShippingAndHandling|Valor do frete e e taxa que você cobra do seu cliente para cobrir os seus custos administrativos de venda|Cybersource|
|ShippingOnly|Valor do frete|Cybersource|
|Subscription|Assinatura. Ex.: Streaming de vídeos / Assinatura de notícias|Cybersource|
