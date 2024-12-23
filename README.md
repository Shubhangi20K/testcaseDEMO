Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'transactionServicesClient' defined in file [D:\Project\SBI\microservices\epay_transaction_service\build\classes\java\main\com\epay\transaction\externalservice\TransactionServicesClient.class]: Failed to instantiate [com.epay.transaction.externalservice.TransactionServicesClient]: No default constructor found
package com.epay.transaction.externalservice;

import com.epay.transaction.dto.DcmsDto;
import com.epay.transaction.dto.EisPayloadDto;
import com.epay.transaction.model.response.EisResponse;
import com.epay.transaction.util.DcmsConstants;
import lombok.AllArgsConstructor;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.web.reactive.function.client.WebClient;

import java.security.SecureRandom;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

@Service
@RequiredArgsConstructor
public class TransactionServicesClient  {

private final WebClient webClient;

   public TransactionServicesClient(WebClient.Builder webclientBuilder){
       this.webClient = webclientBuilder.build();
   }
    public DcmsDto getDcmsDto() {
       return DcmsDto.builder()
               .requestReferenceNumber("SBIDK24156091748082271708")
               .sourceId("DK")
               .destination("DCMS")
               .txntype("LIMIT_FLAG")
               .txnSubType("ENQUIRY")
               .eisPayload(EisPayloadDto.builder()
                       .actionType("INQUIRY")
                       .bankCode("SBI")
                       .sourceId("IN")
                       .build())
               .build();
    }

   public String url = "https://eisuat.sbi.co.in/gen5/gateway/thirdParty/wrapper/services" ;
//    {
//        "REQUEST_REFERENCE_NUMBER": "SBIDK24156091748082271708",
//            "SOURCE_ID": "DK",
//            "DESTINATION": "DCMS",
//            "TXN_TYPE": "LIMIT_FLAG",
//            "TXN_SUB_TYPE": "ENQUIRY",
//            "EIS_PAYLOAD": {
//        "CardNumber": "4591782007414931",
//                "ActionType": "INQUIRY",
//                "BankCode": "SBI",
//                "SourceId": "IN"
//    }
//    }

    public EisResponse sendData(){
       return webClient.post()
               .uri( "https://eisuat.sbi.co.in/gen5/gateway/thirdParty/wrapper/services")
               .bodyValue(getDcmsDto())
               .retrieve()
               .bodyToMono(EisResponse.class)
               .block();

    }

    public static String generateReqRefNo(String sourceId, String clientId) {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("HHmmssSSS");
        String timemill 	= LocalDateTime.now().format(formatter);
        SecureRandom rnd 	= new SecureRandom();
        int number 			= rnd.nextInt(999999);
        String rannumber 	= String.format("%06d", number);
        String formatDayOfYear = DcmsConstants.PATTERN_EMPTY;
        String dayOfYear = String.valueOf(LocalDate.now().getDayOfYear());
        if(!dayOfYear.isEmpty()) {
            formatDayOfYear = String.format("%3s", dayOfYear).replace(' ', '0');
        }

        return sourceId.concat(clientId).concat(String.valueOf(LocalDate.now().getYear() % 100)).concat(formatDayOfYear).concat(timemill).concat(rannumber);

    }

    @Value("${external.api.transaction.services.base.path}")
    public String getCardNuymber(EisPayloadDto eisPayloadDto)
    {


        return eisPayloadDto.toString();
    }
}
