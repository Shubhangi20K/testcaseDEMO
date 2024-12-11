@Service
@RequiredArgsConstructor
public class OrderService {
    LoggerUtility logger = LoggerFactoryUtility.getLogger(OrderService.class);

    private final OrderDao orderDao;
    private final OrderValidator orderValidator;
    private final EncryptionDecryptionUtil encryptionDecryptionUtil;
    private final ObjectMapper objectMapper;
    private final UniqueIDGenerator uniqueIDGenerator;

    public TransactionResponse<OrderResponse> createOrder(OrderRequest orderRequest) {
        //Step 1 : Build orderDTO
        MerchantDto merchantDto = getMerchantDto();
        OrderDto orderDto = buildOrderDTO(orderRequest.getOrderRequest(), merchantDto);
        //Step 2 : Validated orderRequest
        orderValidator.validateOrderRequest(orderDto);
        //Step 3 : Generate Order Hash
        orderDto.setOrderHash(encryptionDecryptionUtil.hashValue(orderDto.getMId(), orderDto.getOrderRefNumber()));
        //Step 4 : Generate SBI Order Reference Number
        orderDto.setSbiOrderRefNumber(uniqueIDGenerator.generateSBIOrderRefNumber());
        //Step 5 : Save Order
        orderDto.setCreatedDate(DateTimeUtils.currentTimeMillis());
        orderDto.setUpdatedDate(DateTimeUtils.currentTimeMillis());
        orderDto = orderDao.saveOrder(orderDto);
        //Step 6 : Build Order Response and send to controller
        return buildTransactionResponse(orderDto, merchantDto);
    }

    public TransactionResponse<OrderResponse> getOrderByOrderRefNumber(String orderRefNumber) {
        MerchantDto merchantDto = getMerchantDto();
        OrderDto orderDto = orderDao.getOrderByOrderRefNumber(orderRefNumber);
        return buildTransactionResponse(orderDto, merchantDto);
    }

    public TransactionResponse<String> updateOrderStatus(String orderRefNumber, String status) {
        orderDao.updateOrderStatus(orderRefNumber, status);
        return TransactionResponse.<String>builder().data(List.of("Order Status updated Successfully.")).status(1).count(1L).build();
    }

    private OrderDto buildOrderDTO(String orderRequest, MerchantDto merchantDto) {
        try {
            String decryptedCustomerRequest = encryptionDecryptionUtil.decryptRequest(orderRequest, merchantDto);
            return objectMapper.readValue(decryptedCustomerRequest, OrderDto.class);
        } catch (JsonProcessingException e) {
            logger.error("Error in buildOrderDTO  ", e);
            throw new TransactionException(ErrorConstants.INVALID_ERROR_CODE, MessageFormat.format(ErrorConstants.INVALID_ERROR_MESSAGE, orderRequest));
        }
    }

    private String buildOrderData(OrderDto orderDto, MerchantDto merchantDto) {
        try {
            return encryptionDecryptionUtil.encryptRequest(objectMapper.writeValueAsString(orderDto), merchantDto);
        } catch (JsonProcessingException e) {
            logger.error("Error in buildCustomerData  ", e);
            throw new TransactionException(ErrorConstants.INVALID_ERROR_CODE, MessageFormat.format(ErrorConstants.INVALID_ERROR_MESSAGE, orderDto));
        }
    }

    private MerchantDto getMerchantDto() {
        EPayPrincipal ePayPrincipal = EPayIdentityUtil.getUserPrincipal();
        return orderDao.getActiveMerchantByMID(ePayPrincipal.getMid()).orElseThrow(() -> new TransactionException(ErrorConstants.NOT_FOUND_ERROR_CODE, MessageFormat.format(ErrorConstants.NOT_FOUND_ERROR_MESSAGE, "Valid Merchant")));
    }

    private TransactionResponse<OrderResponse> buildTransactionResponse(OrderDto orderDto, MerchantDto merchantDto) {
        String orderData = buildOrderData(orderDto, merchantDto);
        OrderResponse orderResponse = OrderResponse.builder().orderRefNum(orderDto.getOrderRefNumber()).orderResponse(orderData).build();
        return TransactionResponse.<OrderResponse>builder().data(List.of(orderResponse)).status(1).count(1L).build();
    }
}


import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.*;
import java.util.Optional;

public class OrderServiceTest {

    @Mock
    private OrderDao orderDao;

    @Mock
    private OrderValidator orderValidator;

    @Mock
    private EncryptionDecryptionUtil encryptionDecryptionUtil;

    @Mock
    private ObjectMapper objectMapper;

    @Mock
    private UniqueIDGenerator uniqueIDGenerator;

    @Mock
    private EPayPrincipal ePayPrincipal;

    @InjectMocks
    private OrderService orderService;

    @BeforeEach
    public void setUp() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    public void testCreateOrder() throws Exception {
        // Arrange
        String orderRequest = "test-order-request";
        MerchantDto merchantDto = new MerchantDto("test-merchant-id");
        OrderDto orderDto = new OrderDto("order-ref-123", "merchant-id-123", 100.0);
        orderDto.setOrderHash("test-order-hash");
        orderDto.setSbiOrderRefNumber("SBI123456");
        
        when(orderDao.getActiveMerchantByMID(anyString())).thenReturn(Optional.of(merchantDto));
        when(orderDao.saveOrder(any(OrderDto.class))).thenReturn(orderDto);
        when(orderValidator.validateOrderRequest(any(OrderDto.class))).thenReturn(true);
        when(encryptionDecryptionUtil.decryptRequest(eq(orderRequest), eq(merchantDto))).thenReturn("{\"orderRefNumber\":\"order-ref-123\",\"merchantId\":\"merchant-id-123\",\"amount\":100.0}");
        when(encryptionDecryptionUtil.hashValue(anyString(), anyString())).thenReturn("test-order-hash");
        when(uniqueIDGenerator.generateSBIOrderRefNumber()).thenReturn("SBI123456");

        // Act
        TransactionResponse<OrderResponse> response = orderService.createOrder(new OrderRequest(orderRequest));

        // Assert
        assertNotNull(response);
        assertEquals(1, response.getStatus());
        assertNotNull(response.getData());
        assertEquals("order-ref-123", response.getData().get(0).getOrderRefNum());
        assertEquals("SBI123456", orderDto.getSbiOrderRefNumber());
    }

    @Test
    public void testGetOrderByOrderRefNumber() {
        // Arrange
        String orderRefNumber = "order-ref-123";
        MerchantDto merchantDto = new MerchantDto("test-merchant-id");
        OrderDto orderDto = new OrderDto(orderRefNumber, "merchant-id-123", 100.0);

        when(orderDao.getOrderByOrderRefNumber(orderRefNumber)).thenReturn(orderDto);

        // Act
        TransactionResponse<OrderResponse> response = orderService.getOrderByOrderRefNumber(orderRefNumber);

        // Assert
        assertNotNull(response);
        assertEquals(1, response.getStatus());
        assertEquals(orderRefNumber, response.getData().get(0).getOrderRefNum());
    }

    @Test
    public void testUpdateOrderStatus() {
        // Arrange
        String orderRefNumber = "order-ref-123";
        String status = "Processed";

        doNothing().when(orderDao).updateOrderStatus(orderRefNumber, status);

        // Act
        TransactionResponse<String> response = orderService.updateOrderStatus(orderRefNumber, status);

        // Assert
        assertNotNull(response);
        assertEquals(1, response.getStatus());
        assertEquals("Order Status updated Successfully.", response.getData().get(0));
    }
}



    public static EPayPrincipal getUserPrincipal() {
       return (EPayPrincipal) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
    }




import static org.mockito.Mockito.*;
import static org.mockito.ArgumentMatchers.anyString;
import static org.mockito.ArgumentMatchers.eq;
import static org.mockito.Mockito.mockStatic;
import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.MockedStatic;

class OrderServiceTest {

    private OrderService orderService;
    private OrderDao orderDao;

    @BeforeEach
    void setUp() {
        orderDao = mock(OrderDao.class);
        orderService = new OrderService(orderDao); // Assuming constructor injection
    }

    @Test
    void testGetOrderByOrderRefNumber_Success() {
        // Mock static method
        try (MockedStatic<EPayIdentityUtil> mockedStatic = mockStatic(EPayIdentityUtil.class)) {
            EPayPrincipal mockPrincipal = mock(EPayPrincipal.class);
            when(mockPrincipal.getMid()).thenReturn("mockMid");
            mockedStatic.when(EPayIdentityUtil::getUserPrincipal).thenReturn(mockPrincipal);

            // Mock other dependencies
            Merchant mockMerchant = new Merchant(); // Assuming this is a valid entity
            when(orderDao.getActiveMerchantByMID(eq("mockMid"))).thenReturn(Optional.of(mockMerchant));

            OrderDto mockOrderDto = new OrderDto(); // Assuming this is a valid entity
            when(orderDao.getOrderByOrderRefNumber(anyString())).thenReturn(mockOrderDto);

            // Execute the method
            TransactionResponse response = orderService.getOrderByOrderRefNumber("mockOrderRef");

            // Verify interactions and assert results
            assertNotNull(response);
            verify(orderDao).getActiveMerchantByMID("mockMid");
            verify(orderDao).getOrderByOrderRefNumber("mockOrderRef");
        }
    }

    @Test
    void testGetOrderByOrderRefNumber_NoMerchantFound() {
        // Mock static method
        try (MockedStatic<EPayIdentityUtil> mockedStatic = mockStatic(EPayIdentityUtil.class)) {
            EPayPrincipal mockPrincipal = mock(EPayPrincipal.class);
            when(mockPrincipal.getMid()).thenReturn("mockMid");
            mockedStatic.when(EPayIdentityUtil::getUserPrincipal).thenReturn(mockPrincipal);

            // Mock other dependencies
            when(orderDao.getActiveMerchantByMID(eq("mockMid"))).thenReturn(Optional.empty());

            // Execute and assert exception
            TransactionException exception = assertThrows(TransactionException.class,
                    () -> orderService.getOrderByOrderRefNumber("mockOrderRef"));

            assertEquals(ErrorConstants.NOT_FOUND_ERROR_CODE, exception.getErrorCode());
            verify(orderDao).getActiveMerchantByMID("mockMid");
            verify(orderDao, never()).getOrderByOrderRefNumber(anyString());
        }
    }
}









    public TransactionResponse<OrderResponse> getOrderByOrderRefNumber(String orderRefNumber) {
        MerchantDto merchantDto = getMerchantDto();
        OrderDto orderDto = orderDao.getOrderByOrderRefNumber(orderRefNumber);
        return buildTransactionResponse(orderDto, merchantDto);
    }

        private MerchantDto getMerchantDto() {
        EPayPrincipal ePayPrincipal = EPayIdentityUtil.getUserPrincipal();
        return orderDao.getActiveMerchantByMID(ePayPrincipal.getMid()).orElseThrow(()
                -> new TransactionException(ErrorConstants.NOT_FOUND_ERROR_CODE,
                MessageFormat.format(ErrorConstants.NOT_FOUND_ERROR_MESSAGE, "Valid Merchant")));
    }



import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private OrderDao orderDao; // Mock the DAO dependency

    @Mock
    private EPayIdentityUtil ePayIdentityUtil; // Mock the utility dependency

    @InjectMocks
    private OrderService orderService; // Inject mocks into the service

    @BeforeEach
    void setUp() {
        // Set up necessary initializations if required
    }

    @Test
    void testGetOrderByOrderRefNumber_Success() {
        // Arrange
        String orderRefNumber = "ORD123";
        MerchantDto mockMerchant = new MerchantDto(); // Mocked MerchantDto
        OrderDto mockOrder = new OrderDto(); // Mocked OrderDto
        EPayPrincipal mockPrincipal = new EPayPrincipal();
        mockPrincipal.setMid("MID123");

        // Mocking static method calls
        Mockito.mockStatic(EPayIdentityUtil.class).when(EPayIdentityUtil::getUserPrincipal).thenReturn(mockPrincipal);

        // Mock the DAO calls
        when(orderDao.getActiveMerchantByMID("MID123")).thenReturn(Optional.of(mockMerchant));
        when(orderDao.getOrderByOrderRefNumber(orderRefNumber)).thenReturn(mockOrder);

        // Act
        TransactionResponse<OrderResponse> response = orderService.getOrderByOrderRefNumber(orderRefNumber);

        // Assert
        assertNotNull(response);
        assertEquals(mockOrder, response.getOrder());
        assertEquals(mockMerchant, response.getMerchant());
    }

    @Test
    void testGetOrderByOrderRefNumber_MerchantNotFound() {
        // Arrange
        String orderRefNumber = "ORD123";
        EPayPrincipal mockPrincipal = new EPayPrincipal();
        mockPrincipal.setMid("MID123");

        Mockito.mockStatic(EPayIdentityUtil.class).when(EPayIdentityUtil::getUserPrincipal).thenReturn(mockPrincipal);

        when(orderDao.getActiveMerchantByMID("MID123")).thenReturn(Optional.empty());

        // Act & Assert
        TransactionException exception = assertThrows(TransactionException.class, 
            () -> orderService.getOrderByOrderRefNumber(orderRefNumber));

        assertEquals(ErrorConstants.NOT_FOUND_ERROR_CODE, exception.getErrorCode());
        assertTrue(exception.getMessage().contains("Valid Merchant"));
    }

    @Test
    void testGetOrderByOrderRefNumber_OrderNotFound() {
        // Arrange
        String orderRefNumber = "ORD123";
        MerchantDto mockMerchant = new MerchantDto();
        EPayPrincipal mockPrincipal = new EPayPrincipal();
        mockPrincipal.setMid("MID123");

        Mockito.mockStatic(EPayIdentityUtil.class).when(EPayIdentityUtil::getUserPrincipal).thenReturn(mockPrincipal);

        when(orderDao.getActiveMerchantByMID("MID123")).thenReturn(Optional.of(mockMerchant));
        when(orderDao.getOrderByOrderRefNumber(orderRefNumber)).thenReturn(null);

        // Act
        TransactionResponse<OrderResponse> response = orderService.getOrderByOrderRefNumber(orderRefNumber);

        // Assert
        assertNotNull(response);
        assertNull(response.getOrder());
        assertEquals(mockMerchant, response.getMerchant());
    }
}

    
