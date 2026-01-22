# TDD Examples: Java

Complete TDD example for Java projects.

## Example: Shopping Cart

### Step 1: RED - Write Failing Test

```java
// ShoppingCartTest.java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import static org.assertj.core.api.Assertions.*;

class ShoppingCartTest {
    private ShoppingCart cart;
    private InMemoryProductCatalog catalog;

    @BeforeEach
    void setUp() {
        catalog = new InMemoryProductCatalog();
        cart = new ShoppingCart(catalog);
    }

    @Test
    @DisplayName("should calculate total for single item")
    void shouldCalculateTotalForSingleItem() {
        catalog.addProduct(new Product("SKU-001", "Widget", 10.00));

        cart.addItem("SKU-001", 2);

        assertThat(cart.getTotal()).isEqualTo(20.00);
    }

    @Test
    @DisplayName("should calculate total for multiple items")
    void shouldCalculateTotalForMultipleItems() {
        catalog.addProduct(new Product("SKU-001", "Widget", 10.00));
        catalog.addProduct(new Product("SKU-002", "Gadget", 25.00));

        cart.addItem("SKU-001", 2);
        cart.addItem("SKU-002", 1);

        assertThat(cart.getTotal()).isEqualTo(45.00);
    }

    @Test
    @DisplayName("should apply percentage discount")
    void shouldApplyPercentageDiscount() {
        catalog.addProduct(new Product("SKU-001", "Widget", 100.00));
        cart.addItem("SKU-001", 1);

        cart.applyDiscount(10); // 10% off

        assertThat(cart.getTotal()).isEqualTo(90.00);
    }

    @Test
    @DisplayName("should throw for unknown product")
    void shouldThrowForUnknownProduct() {
        assertThatThrownBy(() -> cart.addItem("UNKNOWN", 1))
            .isInstanceOf(ProductNotFoundException.class)
            .hasMessageContaining("UNKNOWN");
    }
}
```

Run: `./mvnw test` → Should fail

### Step 2: GREEN - Minimal Implementation

```java
// ShoppingCart.java
import java.util.ArrayList;
import java.util.List;

public class ShoppingCart {
    private final ProductCatalog catalog;
    private final List<CartItem> items = new ArrayList<>();
    private int discountPercent = 0;

    public ShoppingCart(ProductCatalog catalog) {
        this.catalog = catalog;
    }

    public void addItem(String sku, int quantity) {
        Product product = catalog.findBySku(sku)
            .orElseThrow(() -> new ProductNotFoundException(sku));
        items.add(new CartItem(product, quantity));
    }

    public void applyDiscount(int percent) {
        this.discountPercent = percent;
    }

    public double getTotal() {
        double subtotal = items.stream()
            .mapToDouble(item -> item.product().price() * item.quantity())
            .sum();

        return subtotal * (1 - discountPercent / 100.0);
    }
}

record CartItem(Product product, int quantity) {}
record Product(String sku, String name, double price) {}
```

### Step 3: Test Doubles

```java
// InMemoryProductCatalog.java
import java.util.*;

public class InMemoryProductCatalog implements ProductCatalog {
    private final Map<String, Product> products = new HashMap<>();

    public void addProduct(Product product) {
        products.put(product.sku(), product);
    }

    @Override
    public Optional<Product> findBySku(String sku) {
        return Optional.ofNullable(products.get(sku));
    }
}

// ProductCatalog.java
public interface ProductCatalog {
    Optional<Product> findBySku(String sku);
}

// ProductNotFoundException.java
public class ProductNotFoundException extends RuntimeException {
    public ProductNotFoundException(String sku) {
        super("Product not found: " + sku);
    }
}
```

---

## JUnit 5 Patterns

### Nested Tests

```java
@DisplayName("UserService")
class UserServiceTest {

    @Nested
    @DisplayName("createUser")
    class CreateUser {
        @Test
        @DisplayName("should create user with valid email")
        void shouldCreateUserWithValidEmail() { ... }

        @Test
        @DisplayName("should reject invalid email")
        void shouldRejectInvalidEmail() { ... }
    }

    @Nested
    @DisplayName("deleteUser")
    class DeleteUser {
        @Test
        @DisplayName("should remove user")
        void shouldRemoveUser() { ... }
    }
}
```

### Parametrized Tests

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.*;

@ParameterizedTest
@CsvSource({
    "2, 3, 5",
    "0, 0, 0",
    "-1, 1, 0",
    "100, -50, 50"
})
void shouldAdd(int a, int b, int expected) {
    Calculator calc = new Calculator();
    assertThat(calc.add(a, b)).isEqualTo(expected);
}

@ParameterizedTest
@MethodSource("emailProvider")
void shouldValidateEmail(String email, boolean expected) {
    assertThat(isValidEmail(email)).isEqualTo(expected);
}

static Stream<Arguments> emailProvider() {
    return Stream.of(
        Arguments.of("user@example.com", true),
        Arguments.of("invalid", false),
        Arguments.of("@example.com", false)
    );
}
```

### Lifecycle Methods

```java
class DatabaseTest {
    private static Database db;

    @BeforeAll
    static void setUpAll() {
        db = Database.createTestInstance();
    }

    @AfterAll
    static void tearDownAll() {
        db.shutdown();
    }

    @BeforeEach
    void setUp() {
        db.clear();
    }

    @AfterEach
    void tearDown() {
        db.rollback();
    }
}
```

---

## Test Doubles in Java

### Fake (In-Memory Implementation)

```java
public class InMemoryUserRepository implements UserRepository {
    private final Map<String, User> users = new ConcurrentHashMap<>();

    @Override
    public Optional<User> findById(String id) {
        return Optional.ofNullable(users.get(id));
    }

    @Override
    public void save(User user) {
        users.put(user.getId(), user);
    }

    @Override
    public void delete(String id) {
        users.remove(id);
    }
}
```

### Stub (Returns Canned Data)

```java
public class StubPaymentGateway implements PaymentGateway {
    private final boolean shouldSucceed;

    public StubPaymentGateway(boolean shouldSucceed) {
        this.shouldSucceed = shouldSucceed;
    }

    @Override
    public PaymentResult charge(double amount) {
        return shouldSucceed
            ? PaymentResult.success()
            : PaymentResult.failure("Declined");
    }
}
```

### Spy (Records Calls)

```java
public class SpyEmailService implements EmailService {
    private final List<Email> sentEmails = new ArrayList<>();

    @Override
    public void send(String to, String subject, String body) {
        sentEmails.add(new Email(to, subject, body));
    }

    public List<Email> getSentEmails() {
        return Collections.unmodifiableList(sentEmails);
    }

    public record Email(String to, String subject, String body) {}
}

// In test
@Test
void shouldSendWelcomeEmail() {
    SpyEmailService spy = new SpyEmailService();
    NotificationService service = new NotificationService(spy);

    service.sendWelcome("user@example.com");

    assertThat(spy.getSentEmails()).hasSize(1);
    assertThat(spy.getSentEmails().get(0).to()).isEqualTo("user@example.com");
}
```

### Mock with Mockito

```java
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    @Mock
    private PaymentGateway paymentGateway;

    @Mock
    private OrderRepository orderRepository;

    @InjectMocks
    private OrderService orderService;

    @Test
    void shouldChargeCorrectAmount() {
        Order order = new Order("1", 100.00);
        when(orderRepository.findById("1")).thenReturn(Optional.of(order));
        when(paymentGateway.charge(100.00)).thenReturn(PaymentResult.success());

        orderService.processOrder("1");

        verify(paymentGateway).charge(100.00);
        verify(orderRepository).save(argThat(o -> o.getStatus().equals("paid")));
    }
}
```

---

## Commands

```bash
# Run all tests
./mvnw test

# Run specific test class
./mvnw test -Dtest=ShoppingCartTest

# Run specific test method
./mvnw test -Dtest=ShoppingCartTest#shouldCalculateTotalForSingleItem

# Run tests matching pattern
./mvnw test -Dtest=*Cart*

# Skip tests
./mvnw install -DskipTests

# Run with coverage (JaCoCo)
./mvnw test jacoco:report

# Run integration tests
./mvnw verify

# Verbose output
./mvnw test -X
```

### Maven Surefire Configuration

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.2.5</version>
    <configuration>
        <includes>
            <include>**/*Test.java</include>
            <include>**/*Tests.java</include>
        </includes>
        <excludes>
            <exclude>**/*IntegrationTest.java</exclude>
        </excludes>
    </configuration>
</plugin>
```
