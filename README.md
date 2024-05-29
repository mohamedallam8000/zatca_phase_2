# Zatca integration notes
# contact with me on whatsapp : 00201210407130
### Zatca Integration have two steps first : OnBoarding Step example : 
```php
    use Allam\Zatca\OnBoarding;
    $response = (new OnBoarding())
    ->setZatcaEnv('developer-portal')
    ->setZatcaLang('en')
    ->setEmailAddress('email@gmail.com')
    ->setCommonName('TSTCO')
    ->setCountryCode('SA')
    ->setOrganizationUnitName('TSTCO-SA')
    ->setOrganizationName('TSTCO-SA')
    ->setEgsSerialNumber('1-SDSA|2-FGDS|3-SDFG')
    ->setVatNumber('300000000000003')
    ->setInvoiceType('1100')
    ->setRegisteredAddress('SAUDI')
    ->setAuthOtp('111222')
    ->setBusinessCategory('Transportations')
    ->getAuthorization();
```
- you can print previous response as you want to handle it into your system

*******************
### Zatca Integration have two steps second : Send Invoices to zatca Step example :
#### before send invoice important : 
create zatca_documents table
```php
 CREATE TABLE `zatca_documents` (
  `id` bigint(20) UNSIGNED NOT NULL,
  `icv` varchar(191) NOT NULL,
  `uuid` varchar(191) NOT NULL,
  `hash` varchar(191) NOT NULL,
  `xml` longtext DEFAULT NULL,
  `sent_to_zatca` tinyint(1) NOT NULL,
  `sent_to_zatca_status` varchar(191) NOT NULL,
  `signing_time` datetime DEFAULT NULL,
  `response` longtext DEFAULT NULL,
  `invoice_id` varchar(191) NOT NULL,
  `company_id` varchar(191) NOT NULL,
)
```
```php
1-  will initiate record into zatca documents will have auto icv value ex: 1
2-   then check this record is first record or not
3-  it not prevoius hash value will be $prevoius_hash = base64_encode(hash('sha265','0',true));
4-  else initiated recored have prevoius record $prevoius_hash = $prevRecord->hash;
5-  then after invoice is sent will get hash , xml , response and status then update initiated record
6 - generate uuid value and store it for initiated recored and it will be used below
```
```php
    use Allam\Zatca\Invoice\Client;
    use Allam\Zatca\Invoice\Supplier;
    use Allam\Zatca\Invoice\Delivery;
    use Allam\Zatca\Invoice\PaymentType;
    use Allam\Zatca\Invoice\PIH;
    use Allam\Zatca\Invoice\ReturnReason;
    use Allam\Zatca\Invoice\BillingReference;
    use Allam\Zatca\Invoice\AdditionalDocumentReference;
    use Allam\Zatca\Invoice\LegalMonetaryTotal;
    use Allam\Zatca\Invoice\TaxesTotal;
    use Allam\Zatca\Invoice\TaxSubtotal;
    use Allam\Zatca\Invoice\LineTaxCategory;
    use Allam\Zatca\Invoice\InvoiceLine;
    use Allam\Zatca\Invoice\AllowanceCharge;
    use Allam\Zatca\Invoice\InvoiceGenerator;

    $client = (new Client())
    ->setVatNumber('300000000000003')
    ->setStreetName('STREET')
    ->setBuildingNumber('1111')
    ->setPlotIdentification('2223')
    ->setSubDivisionName('JEDDAH')
    ->setCityName('JEDDAH')
    ->setPostalNumber('12222')
    ->setCountryName('SA')
    ->setClientName('TSTCO');

    $supplier = (new Supplier())
    ->setCrn('1000000000')
    ->setStreetName('RIYADH')
    ->setBuildingNumber('2322')
    ->setPlotIdentification('2223')
    ->setSubDivisionName('RIYADH')
    ->setCityName('RIYADH')
    ->setPostalNumber('11633')
    ->setCountryName('SA')
    ->setVatNumber('300000000000003')
    ->setVatName('TSTCO');

    $delivery = (new Delivery())
    ->setDeliveryDateTime('2022-09-07');

    $paymentType = (new PaymentType())
    ->setPaymentType('10');

    $returnReason = (new ReturnReason())
    ->setReturnReason('SET_RETURN_REASON');

    $previous_hash = (new PIH())
    ->setPIH('X+zrZv/IbzjZUnhsbWlsecLbwjndTpG0ZynXOif7V+k=');  // note this value it from step 3 , 4

    $billingReference = (new BillingReference())
    ->setBillingReference('23'); // note this used when type credit or debit this value of parent invoice id

    $additionalDocumentReference = (new AdditionalDocumentReference())
    ->setInvoiceID('55'); // note this value it from step 1

    $legalMonetaryTotal = (new LegalMonetaryTotal())
    ->setTotalCurrency('SAR')
    ->setLineExtensionAmount(4)
    ->setTaxExclusiveAmount(4)
    ->setTaxInclusiveAmount(4.60)
    ->setAllowanceTotalAmount(0)
    ->setPrepaidAmount(0)
    ->setPayableAmount(4.60);

    $taxesTotal = (new TaxesTotal())
    ->setTaxCurrencyCode('SAR')
    ->setTaxTotal(0.60);

    $taxSubtotal = (new TaxSubtotal())
    ->setTaxCurrencyCode('SAR')
    ->setTaxableAmount(4.00)
    ->setTaxAmount(0.60)
    ->setTaxCategory('S')
    ->setTaxPercentage(15)
    ->getElement();

    $itemTaxCategory = (new LineTaxCategory())
    ->setTaxCategory('S')
    ->setTaxPercentage(15)
    ->getElement();

    $invoiceLine = (new InvoiceLine())
    ->setLineID('1')
    ->setLineName('TST Item')
    ->setLineCurrency('SAR')
    ->setLinePrice(2)
    ->setLineQuantity(2)
    ->setLineSubTotal(4)
    ->setLineTaxTotal(0.60)
    ->setLineNetTotal(4.60)
    ->setLineTaxCategories($itemTaxCategory)
    ->setLineDiscountReason('reason')
    ->setLineDiscountAmount(0)
    ->getElement();

    $allowanceCharge = (new AllowanceCharge())
    ->setAllowanceChargeCurrency('SAR')
    ->setAllowanceChargeIndex('1')
    ->setAllowanceChargeAmount(0)
    ->setAllowanceChargeTaxCategory('S')
    ->setAllowanceChargeTaxPercentage(15)
    ->getElement();

    $response = (new InvoiceGenerator())
    ->setZatcaEnv('developer-portal')
    ->setZatcaLang('en')
    ->setInvoiceNumber('SME00023')
    ->setInvoiceUuid('8d487816-70b8-4ade-a618-9d620b73814a') // this value from step 6
    ->setInvoiceIssueDate('2022-09-07')
    ->setInvoiceIssueTime('12:21:28')
    ->setInvoiceType('0200000','388')
    ->setInvoiceCurrencyCode('SAR')
    ->setInvoiceTaxCurrencyCode('SAR')
    //->setInvoiceBillingReference($billingReference)  use this when document type is credit or debit
    ->setInvoiceAdditionalDocumentReference($additionalDocumentReference)
    ->setInvoicePIH($previous_hash)
    ->setInvoiceSupplier($supplier)
    ->setInvoiceClient($client)
    ->setInvoiceDelivery($delivery)
    ->setInvoicePaymentType($paymentType)
    //->setInvoiceReturnReason($returnReason) use this when document type is credit or debit
    ->setInvoiceLegalMonetaryTotal($legalMonetaryTotal)
    ->setInvoiceTaxesTotal($taxesTotal)
    ->setInvoiceTaxSubTotal($taxSubtotal)
    ->setInvoiceAllowanceCharges($allowanceCharge)
    ->setInvoiceLines($invoiceLine)
    ->setCertificateEncoded("TUlJQjVUQ0NBWXFnQXdJQkFnSUdBWStPTTBOR01Bb0dDQ3FHU000OUJBTUNNQlV4RXpBUkJnTlZCQU1NQ21WSmJuWnZhV05wYm1jd0hoY05NalF3TlRFNU1EQXhORE13V2hjTk1qa3dOVEU0TWpFd01EQXdXakJETVE0d0RBWURWUVFEREFWVVUxUkRUekVSTUE4R0ExVUVDd3dJVkZOVVEwOHRVMEV4RVRBUEJnTlZCQW9NQ0ZSVFZFTlBMVk5CTVFzd0NRWURWUVFHRXdKVFFUQldNQkFHQnlxR1NNNDlBZ0VHQlN1QkJBQUtBMElBQkFsbnRVditjUkFJU0JSekFKTWFSUHdrRE5JblZKdGNXV3l1UWdYN0k2U0s0QytTSU1JQ0psYzN2YXhkYUpQc2pRUlJ4VHE3eDZCbnZHS09JUTVMdDNLamdab3dnWmN3REFZRFZSMFRBUUgvQkFJd0FEQ0JoZ1lEVlIwUkJIOHdmYVI3TUhreEhUQWJCZ05WQkFRTUZERXRVMFJUUVh3eUxVWkhSRk44TXkxVFJFWkhNUjh3SFFZS0NaSW1pWlB5TEdRQkFRd1BNekF3TURBd01EQXdNREF3TURBek1RMHdDd1lEVlFRTURBUXhNVEF3TVE0d0RBWURWUVFhREFWVFFWVkVTVEVZTUJZR0ExVUVEd3dQVkhKaGJuTndiM0owWVhScGIyNXpNQW9HQ0NxR1NNNDlCQU1DQTBrQU1FWUNJUUNIUDZEMDVNRm9rU1lickdNV2RPVzhqL1htU0lwdURwUDRId25IckRxOFFBSWhBTEZ2THg4NGRvUWpaa0U0M1JKZzFXYWdVcm9XQkNpN0kzWk9RdVlCNk9Ibg==")
    ->setPrivateKeyEncoded("LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JR0VBZ0VBTUJBR0J5cUdTTTQ5QWdFR0JTdUJCQUFLQkcwd2F3SUJBUVFnb0pYTGxHRDE4MXZaaFgrUzRDMTQKODRURGVJUWV6dmtKR2l5TkdNZktjck9oUkFOQ0FBUUpaN1ZML25FUUNFZ1Vjd0NUR2tUOEpBelNKMVNiWEZscwpya0lGK3lPa2l1QXZraURDQWlaWE43MnNYV2lUN0kwRVVjVTZ1OGVnWjd4aWppRU9TN2R5Ci0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0K")
    ->setCertificateSecret("srtiZ72Dx+YySBGO22hmr5UEaul5HKl8snfTfUnc/vY=")
    ->sendDocument(); // when you use production certifiacte for (simulation , core) dont forget set sendDocument(true)
```
- important note : returned data update it to initiated row from step 1
