@Component
@RequiredArgsConstructor
public class CustomerDao {

    private final CustomerRepository customerRepository;
    private final MerchantDao merchantDao;
    private final ObjectMapper objectMapper;

    public MerchantDto getActiveMerchantByMID(String mID) {
        return merchantDao.getActiveMerchantByMID(mID);
    }

    public Optional<CustomerDto> findCustomerByEmailPhoneAndMerchant(String email, String phone, String mid) {
        Optional<Customer> customer = customerRepository.findByEmailOrPhoneNumberAndMerchantId(email, phone, mid);
        return customer.map(value -> objectMapper.convertValue(value, CustomerDto.class));
    }

    public CustomerDto saveCustomer(CustomerDto customerDto) {
        Customer customer = objectMapper.convertValue(customerDto, Customer.class);
        customer = customerRepository.save(customer);
        return objectMapper.convertValue(customer, CustomerDto.class);
    }

    public Optional<CustomerDto> getCustomerByCustomerId(String customerId) {
        Optional<Customer> customer = customerRepository.findByCustomerId(customerId);
        return customer.map(value -> objectMapper.convertValue(value, CustomerDto.class));
    }

    /**
     * Update customer status
     *
     * @param customerDto
     * @return CustomerDto
     */
    public CustomerDto updateCustomerStatus(CustomerDto customerDto) {
        Customer customer = objectMapper.convertValue(customerDto, Customer.class);
        customerRepository.save(customer);
        return objectMapper.convertValue(customer, CustomerDto.class);
    }
}




import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.MockitoAnnotations;

import java.util.Optional;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

public class YourServiceTest {

    @Mock
    private CustomerRepository customerRepository;

    @Mock
    private MerchantDao merchantDao;

    @Mock
    private ObjectMapper objectMapper;

    @InjectMocks
    private YourService yourService;  // Replace YourService with the actual class name

    @BeforeEach
    public void setup() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    public void testGetActiveMerchantByMID() {
        String mid = "12345";
        MerchantDto mockMerchantDto = new MerchantDto();
        when(merchantDao.getActiveMerchantByMID(mid)).thenReturn(mockMerchantDto);

        MerchantDto result = yourService.getActiveMerchantByMID(mid);

        assertNotNull(result);
        assertEquals(mockMerchantDto, result);
        verify(merchantDao, times(1)).getActiveMerchantByMID(mid);
    }

    @Test
    public void testFindCustomerByEmailPhoneAndMerchant() {
        String email = "test@example.com";
        String phone = "1234567890";
        String mid = "12345";
        Customer mockCustomer = new Customer();
        CustomerDto mockCustomerDto = new CustomerDto();

        when(customerRepository.findByEmailOrPhoneNumberAndMerchantId(email, phone, mid))
                .thenReturn(Optional.of(mockCustomer));
        when(objectMapper.convertValue(mockCustomer, CustomerDto.class))
                .thenReturn(mockCustomerDto);

        Optional<CustomerDto> result = yourService.findCustomerByEmailPhoneAndMerchant(email, phone, mid);

        assertTrue(result.isPresent());
        assertEquals(mockCustomerDto, result.get());
        verify(customerRepository, times(1)).findByEmailOrPhoneNumberAndMerchantId(email, phone, mid);
    }

    @Test
    public void testSaveCustomer() {
        CustomerDto customerDto = new CustomerDto();
        Customer mockCustomer = new Customer();
        when(objectMapper.convertValue(customerDto, Customer.class)).thenReturn(mockCustomer);
        when(customerRepository.save(mockCustomer)).thenReturn(mockCustomer);
        when(objectMapper.convertValue(mockCustomer, CustomerDto.class)).thenReturn(customerDto);

        CustomerDto result = yourService.saveCustomer(customerDto);

        assertNotNull(result);
        assertEquals(customerDto, result);
        verify(customerRepository, times(1)).save(mockCustomer);
    }

    @Test
    public void testGetCustomerByCustomerId() {
        String customerId = "C123";
        Customer mockCustomer = new Customer();
        CustomerDto mockCustomerDto = new CustomerDto();

        when(customerRepository.findByCustomerId(customerId)).thenReturn(Optional.of(mockCustomer));
        when(objectMapper.convertValue(mockCustomer, CustomerDto.class)).thenReturn(mockCustomerDto);

        Optional<CustomerDto> result = yourService.getCustomerByCustomerId(customerId);

        assertTrue(result.isPresent());
        assertEquals(mockCustomerDto, result.get());
        verify(customerRepository, times(1)).findByCustomerId(customerId);
    }

    @Test
    public void testUpdateCustomerStatus() {
        CustomerDto customerDto = new CustomerDto();
        Customer mockCustomer = new Customer();

        when(objectMapper.convertValue(customerDto, Customer.class)).thenReturn(mockCustomer);
        when(customerRepository.save(mockCustomer)).thenReturn(mockCustomer);
        when(objectMapper.convertValue(mockCustomer, CustomerDto.class)).thenReturn(customerDto);

        CustomerDto result = yourService.updateCustomerStatus(customerDto);

        assertNotNull(result);
        assertEquals(customerDto, result);
        verify(customerRepository, times(1)).save(mockCustomer);
    }
}

