     String eisResponse = getWebClient()
                .post()
                .uri(url)
                .accept(MediaType.valueOf(MediaType.APPLICATION_JSON_VALUE))
               // .headers(headers->headers.add(HttpHeaders.AUTHORIZATION, "AccessToken " + encryptedPublicKey))
                .headers(headers->headers.add(HttpHeaders.AUTHORIZATION, encryptedPublicKey))
                .contentType(MediaType.valueOf(MediaType.APPLICATION_JSON_VALUE))
                .bodyValue(ResponseData1)
                .retrieve()
                .bodyToMono(String.class)  // Deserialize the response to GstResponse
                .block();




					URL myUrl = new URL(null, url);
					HttpURLConnection conn = (HttpURLConnection) myUrl.openConnection();
					
					conn.setRequestProperty("Content-Type", "application/json");
					conn.setRequestProperty("Accept", "application/json");
					conn.setRequestMethod("POST");
					conn.setDoOutput(true);
					
					conn.setRequestProperty("AccessToken", AESKeyusingRSAEncrption.replaceAll("\n", ""));
					BufferedOutputStream bw = new BufferedOutputStream(conn.getOutputStream());
					DataOutputStream wr = new DataOutputStream(bw);
					wr.write(requestData.getBytes("UTF-8"));
					wr.flush();
					wr.close();
					bw.close();
					
