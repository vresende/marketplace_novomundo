##Marketplace NovoMundo
######plataforma: Vtex

* Links Importantes:
     * [Documentação](https://help.vtex.com/tutorial/integration-guide-for-marketplaces-seller-non-vtex--yMji0ow0rQuYgQsg26Kus#a1)

### Passo a passo
> A integração junto a vtex acontece de duas formas. 
>Fornecendo uma API REST para que o Marketplace possa
>* Consultar preço e estoque
>* Receber novos pedidos
>* Receber pedidos aprovados
>* Consulta de Frete
>Abaixo será possível ver cada uma das etapas enumeradas
>
>Em contra partida precisamos consumir a API da Vtex nos seguintes casos
>* Notificar alteração de preço
>* Cadastrar novo SKU
>* Atualizar dados dos pedidos:
>  * Nota Fiscal.
>  * Cancelar pedido sem nota fiscal.
>  * Alterar status do pedido.
>

###Construção da Api.
###### Antes de entrar no fluxo da Vtex
- [ ] Criar tabela em dev..tbl_products_novomundo
- [ ] Ler a tabela dev..products_products para saber quais produtos devem ser cadastrados.
- [ ] Validar poítica de badList.
- [ ] Validar política de marketplace ativo.
- [ ] Criar rotina para ler a tabela
- [ ] Adicionar tabela inventario..tbl_regua_anuncio as atualizações de preço e estoque

##### Notificar Alteração de Valor e estoque:
>   * Sempre que houver uma alteração de preço e estoque, chamar o endpoint de notificação
>da VTEX `https://{AccountName}.vtexcommercestable.com.br/api/catalog_system/pvt/skuSeller/changenotification/[idSeller]/[idskuSeller]`
>   * Se retornar `404` chamar a rotina para criar SKU

#####Inserir SKU:
   * Essa rotina deverá ser chamada em dois momentos:
     * Quando a rotina de notificação de um produto retornar 404
     * Rotina que verifica SKU novos para criação
   * O Que a rotina deve fazer:
     - [ ] Verificar se o codLoja está dentro de dev..vw_Products_Product
     - [ ] Verificar se o produto consta na badlist
     - [ ] Montar json conforme modelo
     - [ ] Enviar o json para o endpont `https://api.vtex.com/{accountName}/suggestions/{sellerId}/{sellerskuid}` 

##### Atualização de preço e estoque
* Iremos receber uma requisição via get com url_encode, `purchaseContext={"items":[{"id":"2002129","quantity":1,"seller":"1"}],"marketingData":null, "postalCode":"22011050","country":"BRA","selectedSla":null,"clientProfileData":null,"geoCoordinates":[]}&sc=1&an=shopfacilfastshop` 
- [ ] Validade JSON ABAIXO
`````
{
	"items": [ //pode vir um array vazio caso item insidponivel
	    {
	        "id": "287611",// obrigatório, string - identificador so SKU
	        "requestIndex": 0, // obrigatório, int - representa a posição desse item no array original (request)
	        "price": 7390, // obrigatório, int - preço por, os dois dígitos menos significativos são os centavos
	        "listPrice": 7490, // obrigatório, int - preço de, os dois dígitos menos significativos são os centavos
	        "quantity": 1, // obrigatório, int - retornar a quantidade solicitada ou a quantidade que consegue atender
	        "seller": "1", // obrigatório, string - retonar o que foi passado no request
	        "priceValidUntil": "2014-03-01T22:58:28.143"  // pode ser nulo, string - data de validade do preço.
	        "offerings":[  //array opcional de ofertas, porém não pode ser nulo: enviar array vazio.
	            {
	                "type":"Garantia", //obrigatório, string - tipo do serviço
	                "id":"5",  //obrigatório, string - identificador do serviço
	                "name":"Garantia de 1 ano", //obrigatório, string - nome do serviço
	                "price":10000  //obrigatório, int - preço do serviço, os dois dígitos menos significativos são os centavos
	            },
	            {
	                "type":"Embalagem de Presente",
	                "id":"6",
	                "name":"Embalagem de Presente",
	                "price":250
	            }
	        ]
	    },
	    {
	        "id": "5837",
	        "requestIndex": 1,
	        "price": 890,
	        "listPrice": 990,
	        "quantity": 5,
	        "seller": "1",
	        "priceValidUntil": null
	    }
	],
	"logisticsInfo": [ // array de informações, quando produtos indisponíveis retornar vazio []
	    {
	        "itemIndex": 0, // obrigatório, int - index do array de items
	        "stockBalance": 99, // obrigatório, int - estoque que atende
	        "quantity": 1, // obrigatório, int - retornar a quantidade solicitada ou a quantidade que consegue atender
	        "shipsTo": [ "BRA"],  // obrigatório, array de string com as siglas dos países de entrega
	        "slas": [  // obrigatório,  pode ser um array vazio na ausencia de CEP.
	            {
	                "id": "Expressa",  //obrigatório, string - identificador tipo entrega
	                "name": "Entrega Expressa",// obrigatório, string - nome do tipo entrega
	                "shippingEstimate": "2bd", // obrigatório, string - doas estimados para a entrega, bd == "business days"
	                "price": 1000 // obrigatório, int - custo da entrega, os dois dígitos menos significativos são os centavos
	                "availableDeliveryWindows": [  // opcional, janelas de entrega,  podendo ser um array vazio
	                ]
	            },
	            {
	                "id": "Agendada",
	                "name": "Entrega Agendada",
	                "shippingEstimate": "5d",  // d - days, bd -business days
	                "price": 800,
	                "availableDeliveryWindows": [
	                     {
	                        "startDateUtc": "2013-02-04T08:00:00+00:00",  // date, obrigatório se for enviado delivery window
	                        "endDateUtc": "2013-02-04T13:00:00+00:00", // date, obrigatório se for enviado delivery window
	                        "price": 0 // int, obrigatório se for enviado delivery window - o valor adicional da entrega agendada
	                    },
	                ]
	            }
	        ]
	    },
	    {
	        "itemIndex": 1,
	        "stockBalance": 1237,
	        "quantity": 5,
	        "shipsTo": [ "BRA" ],
	        "slas": [
	            {
	                "id": "Normal",
	                "name": "Entrega Normal",
	                "shippingEstimate": "5bd", // bd - business days
	                "price": 200
	            }
	        ]
	    }
	],
	"country":"BRA",   // string, nulo se não enviado
	"postalCode":"22251-030"   // string, nulo se não enviado
}
`````
O Endpoint acima deverá ser retornado sempre que o Marketplace chamar. 
O Marketplace poderá fazer a chamada, informando o CEP (nesse caso retornamos os valores de frete)
ou sem CEP onde mandamos o json sem a iformação conforme orientação acima

###### O que o arquivo deve fazer
- [ ] Validar json
- [ ] Limpar tags e scripts
- [ ] Validar Formato do CEP
- [ ] Buscar dados na API na sp_calcular_frete
- [ ] slas: Um array com o retorno do frete
* Com CEP
  - [ ] Buscar informações do produto na view dev..products_products
  - [ ] offerings deve ser um array vazio
  - [ ] stockBalance: EstoqueIntegra
  


#### Receber um pedido
Ao receber um pedido, receberemos um post do seguinte formato

````
[
  {
    "marketplaceOrderId": "B1010",
    "marketplaceServicesEndpoint": "http://sandboxintegracao.vtexcommercestable.com.br/api/oms/",
    "marketplacePaymentValue": 81788,
    "items": [
      {
        "id": "2000037",
        "quantity": 1,
        "seller": "1",
        "commission": 0,
        "freightCommission": 0,
        "price": 80000,
        "bundleItems": [],
        "itemAttachment": {
          "name": null,
          "content": {}
        },
        "attachments": [],
        "priceTags": [],
        "measurementUnit": null,
        "unitMultiplier": 0,
        "isGift": false
      },
      {
        "id": "34562",
        "quantity": 1,
        "seller": "1",
        "commission": 0,
        "freightCommission": 0,
        "price": 1090,
        "bundleItems": [],
        "itemAttachment": {
          "name": null,
          "content": {}
        },
        "attachments": [],
        "priceTags": [],
        "measurementUnit": null,
        "unitMultiplier": 0,
        "isGift": false
      }
    ],
    "clientProfileData": {
      "id": "clientProfileData",
      "email": "jonasrj@hotmail.com",
      "firstName": "Jonas",
      "lastName": "Freitas",
      "documentType": null,
      "document": "08896581885",
      "phone": " 1133-3232",
      "corporateName": null,
      "tradeName": null,
      "corporateDocument": null,
      "stateInscription": null,
      "corporatePhone": null,
      "isCorporate": false,
      "userProfileId": null
    },
    "shippingData": {
      "id": "shippingData",
      "address": {
        "addressType": "Residencial",
        "receiverName": "Jonas",
        "addressId": "Casa",
        "postalCode": "31550500",
        "city": "BELO HORIZONTE",
        "state": "MG",
        "country": "BRA",
        "street": "RUA PEDRO RIBEIRO DE PAULA",
        "number": "66",
        "neighborhood": "COPACABANA",
        "complement": "casa 05",
        "reference": "casa 05",
        "geoCoordinates": []
      },
      "logisticsInfo": [
        {
          "itemIndex": 0,
          "selectedSla": "Normal",
          "lockTTL": "7d",
          "shippingEstimate": "4bd",
          "price": 498,
          "deliveryWindow": null
        },
        {
          "itemIndex": 1,
          "selectedSla": "Normal",
          "lockTTL": "7d",
          "shippingEstimate": "4bd",
          "price": 2,
          "deliveryWindow": null
        }
      ]
    },
    "paymentData": null,
    "openTextField": null,
    "marketingData": null
  }
]
````

Esse pedido não está pago, para os pedidos pagos veremos no item próximo tópico como a documentação da Vtex Sugere.
* O que precisa ser feito.
- [ ] criar uma tabela em dev chamada dev..tbl_pedidos_novomundo

|Nome|Tipo|Observação|
|---|---|---|
|orderId|int|Número do pedido inserido na tabela, é preciso informar no retorno|
| marketplaceOrderId  | string   | Número do pedido no marketplace  |
| marketplaceServicesEndpoint | string | Url do endpoint |
| marketplacePaymentValue | decimal (10,2)| |
| paymentData | dateTime | data do pagamento, valor inicial = null|
| statusAgt | string | ["new", "approved","registered","invoiced","shipped","delivered","canceled"] |
| openTextField | string | |
| marketingData | dateTime | |
| created_at | dateTime | Data do cadastro|
| updated_at | dateTime | última atualização|

- [ ] criar uma tabela em dev chamada dev..tbl_pedidos_itens_novomundo

|Nome|Tipo|Observação|
|---|---|---|
|marketplaceOrderId| string | campo de relação com a tabela dev..tbl_pedidos_novomundo|
| id | string | nosso codLoja|
|quantity| int| quantidade|
|seller| string| seller informado pela NovoMundo
|comission| decimal (10,2)||
|freightCommission| decimal (10,2)||
|price| decimal (10,2)| preço do produto|
|bundleItems | decimal (10,2)| preço do produto|
|attachments | string | |
|priceTags | string | |
|measurementUnit | string | |
|unitMultiplier | int | |
|isGift | int | boolean |
| created_at | dateTime | Data do cadastro|
| updated_at | dateTime | última atualização|

- [ ] criar uma tabela em dev chamada dev..tbl_pedidos_cliente_novomundo
 
 |Nome|Tipo|Observação|
 |---|---|---|
 |marketplaceOrderId| string | campo de relação com a tabela dev..tbl_pedidos_novomundo|
 |id|string| Id do cliente informado pela NovoMundo|
 |email|string||
 |firstName|string||
 |lastName|string||
 |documentType|string||
 |document|string||
 |phone|string||
 |corporateName|string||
 |tradeName|string||
 |corporateDocument|string||
 |stateInscription|string||
 |corporatePhone|string||
 |isCorporate|int| boolean|
 |userProfileId|int| boolean|
 | created_at | dateTime | Data do cadastro|
 | updated_at | dateTime | última atualização|
 
- [ ] criar uma tabela em dev chamada dev..tbl_pedidos_shippingData_novomundo
 
 |Nome|Tipo|Observação|
 |---|---|---|
 |marketplaceOrderId| string | campo de relação com a tabela dev..tbl_pedidos_novomundo|
 |id|string| Id do cliente informado pela NovoMundo|
 |addressType|string||
 |receiverName|string||
 |addressId|string||
 |postalCode|string||
 |city|string||
 |state|string||
 |country|string||
 |street|string||
 |number|string||
 |neighborhood|string||
 |complement|string||
 |reference|string||
 |geoCoordinates|string||
| created_at | dateTime | Data do cadastro|
| updated_at | dateTime | última atualização|
 
 - [ ] criar uma tabela em dev chamada dev..tbl_pedidos_logisticsInfo_novomundo
 
 |Nome|Tipo|Observação|
 |---|---|---|
 |marketplaceOrderId| string | campo de relação com a tabela dev..tbl_pedidos_novomundo|
 |itemIndex|int||
 |selectedSla|string| dias corridos|
 |shippingEstimate|string| dias úteis|
 |shippingEstimate|string| dias úteis|
 |price|decimal (10,2)| dias úteis|
 |deliveryWindow|string ||
 | created_at | dateTime | Data do cadastro|
 | updated_at | dateTime | última atualização|
 
 - [ ] Inserir as informações recebidas em suas devidas tabelas
 - [ ] gerar o json abaixo como retorno com status http 200
 ````
[
  {
    "marketplaceOrderId": "959311095",
    "orderId": "123543123", //** - identificador do pedido inserido no Seller
    "followUpEmail": "75c70c09dbb3498c9b3bbdee59bf0687@ct.vtex.com.br",
    "items": [
      {
        "id": "2002495",
        "quantity": 1,
        "Seller": "1",
        "commission": 0,
        "freightCommission": 0,
        "price": 9990,
        "bundleItems": [],
        "priceTags": [],
        "measurementUnit": "un",
        "unitMultiplier": 1,
        "isGift": false
      }
    ],
    "clientProfileData": {
      "id": "clientProfileData",
      "email": "5c77abaea35f4cb6824b9326942c00e5@ct.vtex.com.br",
      "firstName": "JONAS",
      "lastName": "ALVES DE OLIVEIRA",
      "documentType": "cpf",
      "document": "32133239851",
      "phone": "1592712979",
      "corporateName": null,
      "tradeName": null,
      "corporateDocument": null,
      "stateInscription": null,
      "corporatePhone": null,
      "isCorporate": false,
      "userProfileId": null
    },
    "shippingData": {
      "id": "shippingData",
      "address": {
        "addressType": "Residencial",
        "receiverName": "JONAS ALVES DE OLIVEIRA",
        "addressId": "Casa",
        "postalCode": "13476103",
        "city": "Americana",
        "state": "SP",
        "country": "BRA",
        "street": "JOÃO DAMÁZIO GOMES",
        "number": "121",
        "neighborhood": "SÃO JOSÉ",
        "complement": null,
        "reference": "Bairro Praia Azul / Posto de Saúde 17",
        "geoCoordinates": []
      },
      "logisticsInfo": [
        {
          "itemIndex": 0,
          "selectedSla": "Normal",
          "lockTTL": "8d",
          "shippingEstimate": "5d",
          "price": 1090,
          "deliveryWindow": null
        }
      ]
    },
   "paymentData":null
  }
]
 ````
 Quando houver um erro retornar no seguinte formato com código http 400
 ````
{
	"error": {
	"code": "1",
	"message": "O verbo 'GET' não é compatível com a rota '/api/fulfillment/pvt/orders'",
	"exception": null
	}
}
 ````
 #### Confirmação de pagamento do pedido
  O Novo Mundo irá enviar um post para a url: `https://{Sellerendpoint}/pvt/orders/[orderid]/fulfill?sc={PoliticaComercial}&an= {AccountName}`
  
  com o seguinte body
  ````
{
	"marketplaceOrderId": "959311095" // identificador do pedido originado no Marketplace
}
  ````
Ao receber, precisamos fazer o update na tabela dev..tbl_pedidos_novomundo
e retornar como resposta
````
{
	"date": "2014-10-06 18:52:00",
	"marketplaceOrderId": "959311095",
	"orderId": "123543123", //Id do pedido na tabela
	"receipt": "e39d05f9-0c54-4469-a626-8bb5cff169f8",
}
````
 #### Cadastrar pedido na tabela dev..marketplace_pedido_integra
 O que o arquivo deve fazer:
 - [ ] Ler todos pedidos aprovados na tabela dev..tbl_pedidos_novomundo e cadastrar na tabela dev..tbl_marketplace_pedido_integra
 - [ ] atualizar o `statusAgt` do pedido na tabela dev..tbl_pedidos_novomundo para __registered__
 
 #### Informar dados de Nota Fiscal
 Fazer uma chamada post para `https://{AccountName}.vtexcommercestable.com.br/api/oms/pub/orders/ {marketplaceorderId}/invoice`
 > O campo __orderId__ não é o carrinho, mas sim o id dele na `dev..tbl_pedidos_novomundo`

>Validar com o ivan se consideramos esse número como OrderNumber na tabela dev..tbl_marketplace_pedido_integra

 O que o arquivo deve fazer:
- [ ] Ler a tabela dev..tbl_pedidos_novomundo onde o campo `statusAgt` == `approved`
- [ ] Enviar o json para o endpoint informado acima. O Corpo do post deverá conter um json no seguinte formato
````
{
    "type": "Output", // Output|Input (venda|devolução)
    "invoiceNumber": "NFe-00001", // numero da nota fiscal
    "courier": "", // quando é nota fiscal, dados de tracking vem vazio
    "trackingNumber": "", // quando é nota fiscal, dados de tracking vem vazio
    "trackingUrl": "",// quando é nota fiscal, dados de tracking vem vazio
    "items": [ // itens da nota
      {
        "id": "345117",
        "quantity": 1,
        "price": 9003
      }
    ],
    "issuanceDate": "2013-11-21T00:00:00", // data de emissao da nota
    "invoiceValue": 9508 // valor da nota
}
````
- [ ] Atualizar o campo `statusAgt` na tabela `dev..tbl_pedidos_novomundo` para `invoiced`

 #### Informar rastreamento
 
Quando o Seller entregar o pedido para a transportadora, deve informar as informações de rastreamento - endpoint plataforma VTEX.

Endpoint: `https://{AccountName}.vtexcommercestable.com.br/api/oms/pub/orders/ {marketplaceorderId}/invoice Verb: POST
           Content-Type: application/json
           Accept: application/json`
           
O que o arquivo deve fazer: 
- [ ] Ler a tabela `dev..tbl_pedidos_novomundo` onde o campo `statusAgt` for igual a `invoiced`
- [ ] Pegar o rastreamento e preencher o json abaixo: 

````
{
    "type": "Output",
    "invoiceNumber": "NFe-00001",
    "courier": "Correios", // transportadora
    "trackingNumber": "SR000987654321", // identificador de rastreamentor
    "trackingUrl": "http://traking.correios.com.br/sedex/SR000987654321", // url de rastreamento
    "items": [
      {
        "id": "345117",
        "quantity": 1,
        "price": 9003
      }
    ],
    "issuanceDate": "2013-11-21T00:00:00", // formato esperado
    "invoiceValue": 9508
}
````
resposta
````
{
    "date": "2014-02-07T15:22:56.7612218-02:00", // data do recebimento
    "orderId": "123543123",
    "receipt": "38e0e47da2934847b489216d208cfd91" // protocolo gerado confirmando o recebimento do POST (GUID)
}
````
#
###### Para pensar?
> A Nota Fiscal e o Tracking podem ser enviados na mesma chamada, basta preencher todos os dados do POST.
#
### Cancelar pedido:

Uma solicitação de cancelamento pode ser enviada, caso o pedido se encontre em um estado que se permita cancelar, o pedido será cancelado - endpoint plataforma VTEX.

Endpoint: `https://{AccountName}.vtexcommercestable.com.br/api/oms/pvt/orders/ {marketplaceorderId}/cancel`

>Para cancelar um pedido com nota fiscal já informada, enviar uma Nota Fiscal do tipo Input com o valor cheio do pedido.
