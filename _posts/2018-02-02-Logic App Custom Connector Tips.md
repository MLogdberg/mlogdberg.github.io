---
layout: post
title: Logic App Custom Connector WSDL Tips
author: Mattias Lögdberg
tags: [Logic Apps, ARM Template,Logic Apps Custom Connector]
categories: [LogicApps]
image: 
description: 
permalink: /logicapps/custom-connector-wsdl-tips
---

So continue my journey with the **Logic Apps Custom Connector** and this time we are doing some work with the WSDL part of the **Logic Apps Custom Connector** and I thought I would share some tips and trix I've learned and are using when importing and using Connectors with WSDL.
Since the only way to be a good user of a resource like the *Custom Connector* is to understand how we can investigate behaviors, tweak and fix problems and find debug/logs.

## Import Error 
There are several reasons why there will be errors when importing the WSDL and since it's API Management functionality we can (until documentation is up to speed) see the limits on the API Management site:

[https://docs.microsoft.com/en-us/azure/api-management/api-management-api-import-restrictions#wsdl](https://docs.microsoft.com/en-us/azure/api-management/api-management-api-import-restrictions#wsdl)

I encountered the "rare" error with recursive objects, apparently there where a "lazy" coder that just referenced the whole entity in a parent child situation even if the only "needed" field was the *ID*. Anyway removing the recursive element solves the problem,alternatively manipulating the representation could have been done.

## Runtime problems, unexpected behaviors and errors from the backend system
So there are many reasons why there can be errors sent from the backend system but one of the common ones I've found is *System.FormatException: Input string was not in a correct format* and that is due to a element specified as *int* and the value sent in is **null**. Now how can that be a problem?
Since the generation is done by the same logic as in API Management we can import the WSDL in API Management and see how the actual *liquid* template looks like.

Let's look at an example.

With a system I'm sent a link to the WSDL and a sample this to easy integrate with them, testing the sample works fine but importing the WSDL provides alot more than the sample as we are used to there are more fields than we need. So sending in the test file via *postman* works good but the connector does not work.
Let's take a look why, bellow is the sample a fairly easy message with few elements.

``` 
<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <ProcessArticle xmlns="http://ongoingsystems.se/WSI">
      <GoodsOwnerCode>code1234</GoodsOwnerCode>
      <UserName>user1234</UserName>
      <Password>pass1234</Password>
      <art>
        <ArticleOperation>CreateOrUpdate</ArticleOperation>
        <ArticleIdentification>ArticleNumber</ArticleIdentification>
        <ArticleNumber>6553173735310</ArticleNumber>
		<ArticleName>Display Stand</ArticleName>
        <ArticleDescription>Display Stand</ArticleDescription>
        <ArticleUnitCode>St</ArticleUnitCode>
        <IsStockArticle>1</IsStockArticle>
      </art>
    </ProcessArticle>
  </soap:Body>
</soap:Envelope>
```
But when using the *Custom Connector* all the complex structures are mapped: (this is generated via API Management):
```
 <set-body template="liquid">
			<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns="http://ongoingsystems.se/WSI">
				<soap:Body>
					<ProcessArticle>
						<GoodsOwnerCode>{{body.processArticle.goodsOwnerCode}}</GoodsOwnerCode>
						<UserName>{{body.processArticle.userName}}</UserName>
						<Password>{{body.processArticle.password}}</Password>
						<art>
							<ArticleOperation>{{body.processArticle.art.articleOperation}}</ArticleOperation>
							<ArticleIdentification>{{body.processArticle.art.articleIdentification}}</ArticleIdentification>
							<ArticleSystemId>{{body.processArticle.art.articleSystemId}}</ArticleSystemId>
							<ArticleNumber>{{body.processArticle.art.articleNumber}}</ArticleNumber>
							<ArticleName>{{body.processArticle.art.articleName}}</ArticleName>
							<ProductCode>{{body.processArticle.art.productCode}}</ProductCode>
							<BarCode>{{body.processArticle.art.barCode}}</BarCode>
							<SupplierArticleNumber>{{body.processArticle.art.supplierArticleNumber}}</SupplierArticleNumber>
							<ArticleDescription>{{body.processArticle.art.articleDescription}}</ArticleDescription>
							<ArticleUnitCode>{{body.processArticle.art.articleUnitCode}}</ArticleUnitCode>
							<CountryOfOriginCode>{{body.processArticle.art.countryOfOriginCode}}</CountryOfOriginCode>
							<StatisticsNumber>{{body.processArticle.art.statisticsNumber}}</StatisticsNumber>
							<PurchaseCurrencyCode>{{body.processArticle.art.purchaseCurrencyCode}}</PurchaseCurrencyCode>
							<Weight>{{body.processArticle.art.weight}}</Weight>
							<NetWeight>{{body.processArticle.art.netWeight}}</NetWeight>
							<Volume>{{body.processArticle.art.volume}}</Volume>
							<Length>{{body.processArticle.art.length}}</Length>
							<Width>{{body.processArticle.art.width}}</Width>
							<Height>{{body.processArticle.art.height}}</Height>
							<QuantityPerPallet>{{body.processArticle.art.quantityPerPallet}}</QuantityPerPallet>
							<QuantityPerPackage>{{body.processArticle.art.quantityPerPackage}}</QuantityPerPackage>
							<OrderPoint>{{body.processArticle.art.orderPoint}}</OrderPoint>
							<Price>{{body.processArticle.art.price}}</Price>
							<CustomerPrice>{{body.processArticle.art.customerPrice}}</CustomerPrice>
							<PurchasePrice>{{body.processArticle.art.purchasePrice}}</PurchasePrice>
							<IsStockArticle>{{body.processArticle.art.isStockArticle}}</IsStockArticle>
							<ArticleGroup>
								<ArticleGroupOperation>{{body.processArticle.art.articleGroup.articleGroupOperation}}</ArticleGroupOperation>
								<ArticleGroupIdentification>{{body.processArticle.art.articleGroup.articleGroupIdentification}}</ArticleGroupIdentification>
								<ArticleGroupCode>{{body.processArticle.art.articleGroup.articleGroupCode}}</ArticleGroupCode>
								<ArticleGroupName>{{body.processArticle.art.articleGroup.articleGroupName}}</ArticleGroupName>
							</ArticleGroup>
							<ArticleCategory>
								<TypeOperation>{{body.processArticle.art.articleCategory.typeOperation}}</TypeOperation>
								<TypeIdentification>{{body.processArticle.art.articleCategory.typeIdentification}}</TypeIdentification>
								<Code>{{body.processArticle.art.articleCategory.code}}</Code>
								<Name>{{body.processArticle.art.articleCategory.name}}</Name>
							</ArticleCategory>
							<VatCode>
								<VatCodeOperation>{{body.processArticle.art.vatCode.vatCodeOperation}}</VatCodeOperation>
								<VatCodeIdentification>{{body.processArticle.art.vatCode.vatCodeIdentification}}</VatCodeIdentification>
								<VatCode>{{body.processArticle.art.vatCode.vatCode}}</VatCode>
								<VatPercent>{{body.processArticle.art.vatCode.vatPercent}}</VatPercent>
							</VatCode>
							<DangerousGoods>
								<UNNumber>{{body.processArticle.art.dangerousGoods.uNNumber}}</UNNumber>
								<UNIsMarineHazard>{{body.processArticle.art.dangerousGoods.uNIsMarineHazard}}</UNIsMarineHazard>
								<UNIsDangerousGoods>{{body.processArticle.art.dangerousGoods.uNIsDangerousGoods}}</UNIsDangerousGoods>
								<UNPackageType>{{body.processArticle.art.dangerousGoods.uNPackageType}}</UNPackageType>
								<UNTunnelCodes>
{% for item in body.processArticle.art.dangerousGoods.uNTunnelCodes -%}
<UNTunnelCode>{{item}}</UNTunnelCode>
{% endfor -%}
</UNTunnelCodes>
								<UNClassNumber>{{body.processArticle.art.dangerousGoods.uNClassNumber}}</UNClassNumber>
								<UNProperShippingName>
									<Name>{{body.processArticle.art.dangerousGoods.uNProperShippingName.name}}</Name>
									<LanguageCode>{{body.processArticle.art.dangerousGoods.uNProperShippingName.languageCode}}</LanguageCode>
								</UNProperShippingName>
								<UNLabelNumbers>{{body.processArticle.art.dangerousGoods.uNLabelNumbers}}</UNLabelNumbers>
								<DangerousGoodsCoefficient>{{body.processArticle.art.dangerousGoods.dangerousGoodsCoefficient}}</DangerousGoodsCoefficient>
								<EmSCode>{{body.processArticle.art.dangerousGoods.emSCode}}</EmSCode>
							</DangerousGoods>
							<ArticleNames>
{% for item in body.processArticle.art.articleNames -%}
<ArticleName>
        <Language>
            <LanguageCode>{{item.language.languageCode}}</LanguageCode>
        </Language>
        <ArticleName>{{item.articleName}}</ArticleName>
    </ArticleName>
{% endfor -%}
</ArticleNames>
							<ArticleStructureSpecification>
{% for item in body.processArticle.art.articleStructureSpecification -%}
<StructureArticleDefinition>
    <NumberOfItems>{{item.numberOfItems}}</NumberOfItems>
</StructureArticleDefinition>
{% endfor -%}
</ArticleStructureSpecification>
							<MainSupplier>
								<SupplierIdentificationType>{{body.processArticle.art.mainSupplier.supplierIdentificationType}}</SupplierIdentificationType>
								<SupplierOperation>{{body.processArticle.art.mainSupplier.supplierOperation}}</SupplierOperation>
								<SupplierNumber>{{body.processArticle.art.mainSupplier.supplierNumber}}</SupplierNumber>
								<SupplierName>{{body.processArticle.art.mainSupplier.supplierName}}</SupplierName>
								<Address>
									<Name>{{body.processArticle.art.mainSupplier.address.name}}</Name>
									<Address>{{body.processArticle.art.mainSupplier.address.address}}</Address>
									<Address2>{{body.processArticle.art.mainSupplier.address.address2}}</Address2>
									<Address3>{{body.processArticle.art.mainSupplier.address.address3}}</Address3>
									<PostCode>{{body.processArticle.art.mainSupplier.address.postCode}}</PostCode>
									<City>{{body.processArticle.art.mainSupplier.address.city}}</City>
									<TelePhone>{{body.processArticle.art.mainSupplier.address.telePhone}}</TelePhone>
									<Remark>{{body.processArticle.art.mainSupplier.address.remark}}</Remark>
									<Email>{{body.processArticle.art.mainSupplier.address.email}}</Email>
									<MobilePhone>{{body.processArticle.art.mainSupplier.address.mobilePhone}}</MobilePhone>
									<IsEuCountry>{{body.processArticle.art.mainSupplier.address.isEuCountry}}</IsEuCountry>
									<CountryStateCode>{{body.processArticle.art.mainSupplier.address.countryStateCode}}</CountryStateCode>
									<CountryCode>{{body.processArticle.art.mainSupplier.address.countryCode}}</CountryCode>
									<DeliveryInstruction>{{body.processArticle.art.mainSupplier.address.deliveryInstruction}}</DeliveryInstruction>
									<IsVisible>{{body.processArticle.art.mainSupplier.address.isVisible}}</IsVisible>
									<NotifyBySMS>{{body.processArticle.art.mainSupplier.address.notifyBySMS}}</NotifyBySMS>
									<NotifyByEmail>{{body.processArticle.art.mainSupplier.address.notifyByEmail}}</NotifyByEmail>
									<NotifyByTelephone>{{body.processArticle.art.mainSupplier.address.notifyByTelephone}}</NotifyByTelephone>
								</Address>
								<comment>{{body.processArticle.art.mainSupplier.comment}}</comment>
							</MainSupplier>
							<IsGSPCertified>{{body.processArticle.art.isGSPCertified}}</IsGSPCertified>
							<MaxStockDays>{{body.processArticle.art.maxStockDays}}</MaxStockDays>
							<BarCodePackage>{{body.processArticle.art.barCodePackage}}</BarCodePackage>
							<LinkToPicture>{{body.processArticle.art.linkToPicture}}</LinkToPicture>
							<BarCodePallet>{{body.processArticle.art.barCodePallet}}</BarCodePallet>
							<QuantityPerLayer>{{body.processArticle.art.quantityPerLayer}}</QuantityPerLayer>
							<PalletHeight>{{body.processArticle.art.palletHeight}}</PalletHeight>
							<TaricNumbers>
{% for item in body.processArticle.art.taricNumbers -%}
<TaricNumber>
<Country>
    <CountryCode>{{item.country.countryCode}}</CountryCode>
</Country>
<TaricNumber>{{item.taricNumber}}</TaricNumber>
</TaricNumber>
{% endfor -%}
</TaricNumbers>
							<IsObsolete>{{body.processArticle.art.isObsolete}}</IsObsolete>
							<MinDaysToExpiryDate>{{body.processArticle.art.minDaysToExpiryDate}}</MinDaysToExpiryDate>
							<AdditionalStatisticsNumber>{{body.processArticle.art.additionalStatisticsNumber}}</AdditionalStatisticsNumber>
							<CustomsExportConditions>{{body.processArticle.art.customsExportConditions}}</CustomsExportConditions>
							<ArticleColor>
								<ColorCode>{{body.processArticle.art.articleColor.colorCode}}</ColorCode>
								<ColorName>{{body.processArticle.art.articleColor.colorName}}</ColorName>
							</ArticleColor>
							<ArticleSize>
								<SizeCode>{{body.processArticle.art.articleSize.sizeCode}}</SizeCode>
								<SizeName>{{body.processArticle.art.articleSize.sizeName}}</SizeName>
							</ArticleSize>
							<IsSerialNumberArticle>{{body.processArticle.art.isSerialNumberArticle}}</IsSerialNumberArticle>
							<IsBatchArticle>{{body.processArticle.art.isBatchArticle}}</IsBatchArticle>
							<ArticleDefinitionClasses>
								<ArticleDefinitionClassesOperation>{{body.processArticle.art.articleDefinitionClasses.articleDefinitionClassesOperation}}</ArticleDefinitionClassesOperation>
								<Classes>
{% for item in body.processArticle.art.articleDefinitionClasses.classes -%}
<Class>
<Name>{{item.name}}</Name>
<Code>{{item.code}}</Code>
<Comment>{{item.comment}}</Comment>
</Class>
{% endfor -%}
</Classes>
							</ArticleDefinitionClasses>
							<ArticleFreeDecimal1>{{body.processArticle.art.articleFreeDecimal1}}</ArticleFreeDecimal1>
							<ArticleFreeDecimal2>{{body.processArticle.art.articleFreeDecimal2}}</ArticleFreeDecimal2>
						</art>
					</ProcessArticle>
				</soap:Body>
			</soap:Envelope>
		</set-body>
```
And we start running in to problems due to the fact that we are not sending in more data than earlier but the generated xml is much larger and just look at the enormous representation in the GUI:

![Unmodified Connector in GUI](/assets/uploads/2018/02/LogicAppCustomConnectorBeforeEdit.png)

Therefore we get  some unwanted errors since there are some implications added when sending in starts kin complex structures, since I can't really do much about the Logic I need to modify the data sent in, but I can modify the WSDL and reimport it.
So after changing the WSDL to only contain the elements that I needed in my request it looks alot better and the xml message sent are now matching the sample:

Modified definition in the WSDL so the definition is minimal:

```
<s:complexType name="ArticleDefinition">
        <s:sequence>
          <s:element minOccurs="1" maxOccurs="1" name="ArticleOperation" type="tns:ArticleOperation" />
          <s:element minOccurs="1" maxOccurs="1" name="ArticleIdentification" type="tns:ArticleIdentificationType" />
          <s:element minOccurs="0" maxOccurs="1" name="ArticleNumber" type="s:string" />
          <s:element minOccurs="0" maxOccurs="1" name="ArticleName" type="s:string" />
          <s:element minOccurs="0" maxOccurs="1" name="ArticleDescription" type="s:string" />
          <s:element minOccurs="0" maxOccurs="1" name="ArticleUnitCode" type="s:string" />
          <s:element minOccurs="1" maxOccurs="1" name="IsStockArticle" nillable="true" type="s:boolean" />
          <s:element minOccurs="0" maxOccurs="1" name="Weight" type="s:decimal" />
          <s:element minOccurs="0" maxOccurs="1" name="NetWeight" type="s:decimal" />
          <s:element minOccurs="0" maxOccurs="1" name="Volume" type="s:decimal" />
          <s:element minOccurs="0" maxOccurs="1" name="Length" type="s:decimal" />
          <s:element minOccurs="0" maxOccurs="1" name="Width" type="s:decimal" />
          <s:element minOccurs="0" maxOccurs="1" name="Height" type="s:decimal" />
          <s:element minOccurs="0" maxOccurs="1" name="QuantityPerPallet" type="s:int" />
          <s:element minOccurs="0" maxOccurs="1" name="QuantityPerPackage" type="s:int" />
        </s:sequence>
      </s:complexType>
```
Sample from the GUI:

![Unmodified Connector in GUI](/assets/uploads/2018/02/LogicAppCustomConnectorEdited.png)

The request can now be sent to the backend without any problems.
This approach can be used to both detect problems and also understand the behavior of the **Custom Connector** and changeing the WSDL can help us to easier use the Connector even if the maintainance is heavier and we need to keep track of these changes and do the again if a update WSDL would be imported.

## Summary
As stated we need to find the ways of understanding the resource before beeing good users of it, so I hope this will help out and give ideas around workarounds for some troublesome scenarios and that more debug and customization properties is coming along the way.
When more power is needed I would advise using API Management for now until there is more customization properties but most of the time it works like a charm!


