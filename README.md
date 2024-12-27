Pre-Requisites for DCMS API:-
URL:-  
https://eisuat.sbi.co.in/gen5/gateway/thirdParty/wrapper/services

Rquest:- 
{
    "REQUEST_REFERENCE_NUMBER": "SBIDK24156091748082271708",
    "SOURCE_ID": "DK",
    "DESTINATION": "DCMS",
    "TXN_TYPE": "LIMIT_FLAG",			
    "TXN_SUB_TYPE": "ENQUIRY",
    "EIS_PAYLOAD": {
        "CardNumber": "4591782007414931",
        "ActionType": "INQUIRY",
        "BankCode": "SBI",
        "SourceId": "IN"
    }
}

Response:- 
{
    "EIS_RESPONSE": {
        "ChannelDetails": [
            {
                "Flag": "Y",
                "mLimit": "NA",
                "cId": "DOM",
                "eLimit": "NA"
            },
            {
                "Flag": "N",
                "mLimit": "NA",
                "cId": "INT",
                "eLimit": "NA"
            },
            {
                "Flag": "Y",
                "mLimit": "40000",
                "cId": "ATM",
                "eLimit": "000000000000"
            },
            {
                "Flag": "Y",
                "mLimit": "75000",
                "cId": "POS",
                "eLimit": "000000000000"
            },
            {
                "Flag": "N",
                "mLimit": "NA",
                "cId": "ECOM",
                "eLimit": "NA"
            },
