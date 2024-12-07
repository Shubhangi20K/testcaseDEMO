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
