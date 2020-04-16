## SDK .NET para integraÃ§Ã£o com o Boleto FÃ¡cil

Este SDK (Software Development Kit) para o Boleto FÃ¡cil tem como objetivo abstrair, para desenvolvedores de aplicaÃ§Ãµes na plataforma .NET, os detalhes de comunicaÃ§Ã£o com a [API do Boleto FÃ¡cil](https://www.boletobancario.com/boletofacil/integration/integration.html), tanto com o servidor de [produÃ§Ã£o](https://www.boletobancario.com/boletofacil/) como com o servidor de testes ([sandbox](https://sandbox.boletobancario.com/boletofacil/)), de modo que o desenvolvedor possa se concentrar na lÃ³gica de negÃ³cio de sua aplicaÃ§Ã£o.

## Requisitos

* Portable Class Library (.NETFramework 4.5, Windows 8.0)

## IntegraÃ§Ã£o

#### NuGet

O SDK .NET do Boleto FÃ¡cil estÃ¡ disponÃ­vel no NuGet: https://www.nuget.org/packages/boletofacilsdk

## LimitaÃ§Ãµes

O Ãºnico item da API do Boleto FÃ¡cil que essa SDK nÃ£o contempla Ã© a [notificaÃ§Ã£o de pagamentos](https://www.boletobancario.com/boletofacil/integration/integration.html#notificacao) para aplicaÃ§Ãµes Web, atravÃ©s da URL de notificaÃ§Ã£o. Nesse caso, tanto a lÃ³gica de captura das requisiÃ§Ãµes POST enviadas pelo Boleto FÃ¡cil com os dados dos pagamentos como a lÃ³gica da baixa das cobranÃ§as pagas ficam a cargo do sistema integrado com o Boleto FÃ¡cil.

## Guia de uso

Para usar o SDK do Boleto FÃ¡cil Ã© necessÃ¡rio definir dois itens:

1. O ambiente: produÃ§Ã£o (`PRODUCTION`) ou testes (`SANDBOX`)
2. O token do favorecido, o qual deve ser definido na Ã¡rea de **integraÃ§Ã£o** do ambiente escolhido ([aqui](https://www.boletobancario.com/boletofacil/integration/integration.html#token) para produÃ§Ã£o ou [aqui](https://sandbox.boletobancario.com/boletofacil/integration/integration.html#token) para sandbox)

Exemplo:
```c#
// Cria uma instÃ¢ncia do SDK que irÃ¡ enviar requisiÃ§Ãµes ao ambiente de testes do Boleto FÃ¡cil (Sandbox)
BoletoFacil boletoFacil = new BoletoFacil(BoletoFacilEnvironment.SANDBOX, "XYZ12345"); // XYZ12345 is the API key
```

### Gerando uma cobranÃ§a

`Charge` Ã© a classe que representa uma cobranÃ§a do Boleto FÃ¡cil e que contÃ©m os atributos relacionados a ela, que 
sÃ£o exatamente os atributos disponibilizados pela API do Boleto FÃ¡cil e podem ser conferidos [aqui](https://www.boletobancario.com/boletofacil/integration/integration.html#cobrancas). 

Dentre os atributos da cobranÃ§a estÃ£o os dados do pagador, que sÃ£o definidos na classe `Payer`.

```c#
Payer payer = new Payer();
payer.Name = "Pagador teste - SDK .NET";
payer.CpfCnpj = "11122233300";

Charge charge = new Charge();
charge.Description = "CobranÃ§a teste gerada pelo SDK .NET";
charge.Amount = 176.45m;
charge.Payer = payer;

ChargeResponse response = boletoFacil.IssueCharge(charge);
foreach (Charge c in response.Data.Charges)
{
    Console.WriteLine(c);
}
```

A classe `ChargeResponse` indica se a requisiÃ§Ã£o foi bem sucedida ou nÃ£o (da mesma forma que todas as classes que herdam da superclasse `Response` no SDK) e, alÃ©m disso, contÃ©m a lista de cobranÃ§as que foram geradas pela requisiÃ§Ã£o, em uma lista de objetos do tipo `Charge`.


### Consulta de saldo

Por padrÃ£o, as requisiÃ§Ãµes feitas pelo SDK desserializam o retorno em **JSON** para popular os objetos com as informaÃ§Ãµes das requisiÃ§Ãµes, mas o SDK tambÃ©m provÃª a possibilidade de alterar a formataÃ§Ã£o do retorno da API para **XML**, conforme pode ser visto no exemplo abaixo:

```c#
FetchBalanceResponse response = boletoFacil.FetchBalance(ResponseType.XML);
Console.WriteLine(response.Data);
```


### SolicitaÃ§Ã£o de transferÃªncia

Mesmo que se deseje solicitar uma transferÃªncia com o saldo total, Ã© necessÃ¡rio passar um parÃ¢metro da classe `Transfer`, sem o atributo `amount` definido, no caso.

```c#
Transfer transfer = new Transfer();
TransferResponse response = boletoFacil.RequestTransfer(transfer);
Console.WriteLine(response);
```

Como a resposta de solicitaÃ§Ã£o transferÃªncia contÃ©m apenas se a requisiÃ§Ã£o foi bem sucedida ou nÃ£o, nÃ£o se aplica o mÃ©todo `getData()` para ela.


### Consulta de pagamentos e cobranÃ§as

Para esta requisiÃ§Ã£o, Ã© usado um objeto da classe `ListChargesDates` para definir as datas usadas no filtro da consulta. No exemplo abaixo, sÃ£o usadas apenas as datas de vencimento das cobranÃ§as desejadas.

```c#
ListChargesDates dates = new ListChargesDates();
dates.BeginDueDate = DateTime.Today.AddDays(1);
dates.EndDueDate = DateTime.Today.AddDays(5);

ListChargesResponse response = boletoFacil.listCharges(dates);
foreach (Charge c in response.Data.Charges)
{
    Console.WriteLine(c);
}
```


### CriaÃ§Ã£o de favorecido (API AvanÃ§ada)

A API avanÃ§ada tambÃ©m estÃ¡ disponÃ­vel no SDK. Segue abaixo um exemplo de criaÃ§Ã£o de favorecido, com os principais atributos (e objetos) relacionados.

```c#
Person person = new Person();
payee.Name = "Favorecido do SDK .NET";
payee.CpfCnpj = "11122233300";
payee.Email = "email@teste.com";
payee.Password = "senha";
payee.BirthDate = DateTime.Today.AddYears(-18);
payee.Phone = "(41) 91234-4321";
payee.LinesOfBusiness = "Linha de negÃ³cio";
payee.AccountHolder = new Person 
{ 
	Name = "Favorecido do SDK .NET", 
	CpfCnpj = "11122233300" 
};
payee.BankAccount = new BankAccount
{
	BankAccountType = BankAccountType.CHECKING,
    BankNumber = "237",
    AgencyNumber = "123",
    AccountNumber = "4567",
    AccountComplementNumber = 0
};
payee.Category = Category.OTHER;
payee.Address = new Address
{
    Street = "Rua Teste",
    Number = "123",
    City = "4106902",
    State = "PR",
    Postcode = "12345000"
};
payee.BusinessAreaId = 1000;

PayeeResponse response = boletoFacil.createPayee(payee);
if (response.isSuccess()) {
	System.out.println(response.getData());
}
```

A tabela com os cÃ³digos de municÃ­pio do IBGE pode ser consultada [aqui](http://www.ibge.gov.br/home/geociencias/areaterritorial/area.shtm).

### TokenizaÃ§Ã£o do cartÃ£o de crÃ©dito (API AvanÃ§ada)

A tokenizaÃ§Ã£o permite que o salvamento do cartÃ£o de crÃ©dito para compras futuras no padrÃ£o PCI. Segue abaixo um exemplo de tokenizaÃ§Ã£o. 

```c#
Charge charge = new Charge
{
    CreditCardHash = "d6f15405-5870-4e30-8eee-db0f42bf74ce"
};

TokenizationResponse response = boletoFacil.CardTokenization(charge);
Console.WriteLine(response);
```



## AplicaÃ§Ã£o cliente de exemplo

Juntamente com o projeto do SDK hÃ¡ um outro projeto de um cliente de exemplo (aplicaÃ§Ã£o console) que contÃ©m exemplos de todas as chamadas disponibilizadas pelo SDK.


## Suporte

Em caso de dÃºvidas, problemas ou sugestÃµes, nÃ£o hesite em contatar nossa [equipe de implantação](mailto:implantacao@juno.com.br).

