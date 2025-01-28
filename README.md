Last edited by Chaitanya Sonawane 2 weeks ago
Share folder: https://neosofttechnologiesmail-my.sharepoint.com/:f:/g/personal/binoy_medhi_neosofttech_com/En35FMBnFlVNkEv5swpW7fYB0QHK_BFyKospf7bEJdfixA?e=2ytwjQs

   TokenDto tokenDto = new TokenDto();
        tokenDto.setTokenType(TRANSACTION);
        tokenDto.setTokenType(ACCESS);
        tokenDto.setGeneratedToken("GeneratedTok");
        OrderDto orderDto= new OrderDto();
        orderDto.setOrderHash("OrderHash");
        orderDto.setExpiry(455466L);
        orderDto.setMID("mID");
        Optional<TokenDto> optional = Optional.of(tokenDto);
