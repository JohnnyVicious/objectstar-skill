# Objectstar to Java Migration Reference

## Critical Migration Challenges

### 1. Untyped LOCAL Variables

**Problem**: Objectstar LOCAL variables have no declared type—they inherit type dynamically from assigned values.

**Solution**: Analyze usage patterns to infer types.

```
-- Objectstar
LOCAL COUNT, NAME, ACTIVE;
COUNT = 0;           -- Inferred: numeric
NAME = 'John';       -- Inferred: string  
ACTIVE = 'Y';        -- Inferred: logical (boolean)
```

```java
// Java equivalent
int count = 0;
String name = "John";
boolean active = true;
```

**Complex cases**: When a variable is used with multiple types, create a wrapper class or use `Object` with explicit casting.

### 2. Non-Standard Variable Scope

**Problem**: LOCAL variables are visible in declaring rule AND all descendant rules.

```
-- Objectstar
PARENT_RULE;
LOCAL SHARED_VAR;              -- Visible in CHILD_RULE too!
SHARED_VAR = 'set in parent';
CALL CHILD_RULE;

CHILD_RULE;
-- Can read/modify SHARED_VAR without declaration
SHARED_VAR = 'modified in child';
```

**Solution**: Create explicit context objects or use ThreadLocal.

```java
// Java - Context object approach
public class RuleContext {
    private Map<String, Object> locals = new HashMap<>();
    
    public void setLocal(String name, Object value) {
        locals.put(name, value);
    }
    
    public Object getLocal(String name) {
        return locals.get(name);
    }
}

public void parentRule(RuleContext ctx) {
    ctx.setLocal("SHARED_VAR", "set in parent");
    childRule(ctx);  // Pass context explicitly
}

public void childRule(RuleContext ctx) {
    ctx.setLocal("SHARED_VAR", "modified in child");
}
```

### 3. Parameterized Tables

**Problem**: Objectstar tables can be parameterized: `SALES(REGION)` creates logically separate tables per parameter value.

```
-- Objectstar
GET SALES('WEST') WHERE YEAR = 2024;
GET SALES('EAST') WHERE YEAR = 2024;
```

**Solution**: Use composite keys or filtered repositories.

```java
// Java - Composite key approach
@Entity
public class Sales {
    @EmbeddedId
    private SalesId id;  // Contains region + primary key
    
    private int year;
    private BigDecimal amount;
}

// Repository with region filter
public interface SalesRepository {
    List<Sales> findByIdRegionAndYear(String region, int year);
}
```

### 4. Condition Quadrant Logic

**Problem**: Y/N columns with numbered actions have no direct equivalent.

```
-- Objectstar
CUST_TYPE = 'PREMIUM';     | Y N N
ORDER_AMT > 1000;          |   Y N
---------------------------+------
DISCOUNT = 0.20;           | 1
DISCOUNT = 0.10;           |   1
DISCOUNT = 0.05;           |     1
```

**Solution A**: Switch expressions (Java 14+)

```java
double discount = switch (true) {
    case custType.equals("PREMIUM") -> 0.20;
    case orderAmt > 1000 -> 0.10;
    default -> 0.05;
};
```

**Solution B**: Decision table pattern

```java
public class DiscountDecisionTable {
    public double calculate(String custType, BigDecimal orderAmt) {
        if ("PREMIUM".equals(custType)) return 0.20;
        if (orderAmt.compareTo(BigDecimal.valueOf(1000)) > 0) return 0.10;
        return 0.05;
    }
}
```

### 5. TRANSFERCALL (No Return)

**Problem**: TRANSFERCALL transfers control without returning, ending the current transaction.

```
-- Objectstar
TRANSFERCALL NEW_TRANSACTION(params);  -- Never returns here
```

**Solution**: Restructure control flow, use exceptions or explicit state machines.

```java
// Java - State machine approach
public enum TransactionState { CONTINUE, TRANSFER }

public TransactionState currentRule() {
    // ... processing ...
    if (shouldTransfer) {
        nextTransaction = "NEW_TRANSACTION";
        return TransactionState.TRANSFER;
    }
    return TransactionState.CONTINUE;
}

// Main loop
while (state == TransactionState.TRANSFER) {
    state = executeTransaction(nextTransaction);
}
```

## Element Mapping Table

| Objectstar | Java Equivalent |
|------------|-----------------|
| Rule | Method or Service class |
| LOCAL variable | Local variable / context field |
| TDS table | JPA Entity / RDBMS table |
| TEM table | Local object / Map |
| SES table | Session-scoped bean / Redis |
| SCR table | DTO / View model |
| FORALL | for-each loop / Stream |
| GET | Repository.findBy() / SELECT |
| INSERT | Repository.save() / INSERT |
| REPLACE | Repository.save() / UPDATE |
| DELETE | Repository.delete() / DELETE |
| CALL | Method call |
| EXECUTE | @Transactional(propagation=REQUIRES_NEW) |
| SCHEDULE | @Async / Message queue |
| COMMIT | Transaction commit |
| ROLLBACK | Transaction rollback |
| ON exception | try-catch |
| SIGNAL | throw new Exception() |
| DISPLAY | Controller + View |
| LIBRARY | JAR / Maven module |
| Event Rule (V) | Bean Validation / @PrePersist |
| Event Rule (T) | @EntityListeners / Triggers |
| Event Rule (D) | @Formula / Computed property |

## Data Type Mapping

| Objectstar Semantic | Objectstar Syntax | Java Type |
|---------------------|-------------------|-----------|
| Count (C) | B (Binary) | int / long |
| Count (C) | P (Packed) | int / long |
| Quantity (Q) | P (Packed) | BigDecimal |
| Quantity (Q) | F (Float) | double |
| Date (D) | P (Packed) | LocalDate |
| String (S) | C (Char) | String |
| String (S) | V (Variable) | String |
| Identifier (I) | C/V | String |
| Logical (L) | C | boolean |
| Unicode (S) | UN | String |

## Exception Mapping

| Objectstar Exception | Java Equivalent |
|---------------------|-----------------|
| GETFAIL | EmptyResultDataAccessException / Optional.empty() |
| INSERTFAIL | DataIntegrityViolationException |
| REPLACEFAIL | OptimisticLockException |
| DELETEFAIL | DataAccessException |
| LOCKFAIL | PessimisticLockException |
| COMMITLIMIT | TransactionSystemException |
| VALIDATEFAIL | ConstraintViolationException |
| ZERODIVIDE | ArithmeticException |
| CONVERSION | NumberFormatException |
| NULLVALUE | NullPointerException |
| STRINGSIZE | StringIndexOutOfBoundsException |

## Migration Strategies

### Strategy 1: Automated Conversion (Recommended)

Use FreeSoft CodeLiberator or AveriSource for automated conversion:
- Preserves business logic
- Generates maintainable Java
- Handles screen-to-MVC conversion
- Requires validation and testing

### Strategy 2: Rule-by-Rule Rewrite

For smaller codebases or when automated tools aren't available:

1. **Extract MetaStor definitions** (DataLiberator or manual)
2. **Create database schema** from TDS tables
3. **Build entity layer** (JPA entities for each TDS)
4. **Convert rules to services** (one class per logical module)
5. **Migrate screens to MVC** (Spring MVC / Thymeleaf)
6. **Implement exception handling** (try-catch mapping)
7. **Add transaction management** (@Transactional)
8. **Test extensively** (unit + integration tests)

### Strategy 3: Strangler Fig Pattern

For large, critical systems:

1. Build new Java services alongside Objectstar
2. Route new features to Java
3. Gradually migrate existing rules
4. Decommission Objectstar components one by one

## FORALL to Java Patterns

### Simple Iteration
```
-- Objectstar
FORALL EMPLOYEES WHERE DEPT = 'IT' UNTIL GETFAIL:
    CALL PROCESS(EMPLOYEES.EMP#);
END;
```

```java
// Java
employeeRepository.findByDept("IT")
    .forEach(emp -> process(emp.getEmpNo()));
```

### Ordered Iteration
```
-- Objectstar
FORALL EMPLOYEES ORDERED DESCENDING SALARY UNTIL GETFAIL:
    CALL PRINT_EMP(EMPLOYEES.EMP#);
END;
```

```java
// Java
employeeRepository.findAllByOrderBySalaryDesc()
    .forEach(emp -> printEmp(emp.getEmpNo()));
```

### Accumulation
```
-- Objectstar
LOCAL TOTAL;
TOTAL = 0;
FORALL ORDERS WHERE CUST# = target UNTIL GETFAIL:
    TOTAL = TOTAL + ORDERS.AMOUNT;
END;
```

```java
// Java
BigDecimal total = orderRepository.findByCustNo(target)
    .stream()
    .map(Order::getAmount)
    .reduce(BigDecimal.ZERO, BigDecimal::add);
```

## Screen Migration Patterns

### Screen to Spring MVC

```
-- Objectstar Screen Display
UNTIL EXIT_DISPLAY DISPLAY ORDER_SCREEN:
    CALL VALIDATE_INPUT;
    CALL SAVE_ORDER;
END;
```

```java
// Spring MVC Controller
@Controller
@RequestMapping("/orders")
public class OrderController {
    
    @GetMapping("/new")
    public String showOrderForm(Model model) {
        model.addAttribute("order", new OrderDTO());
        return "order-form";  // Thymeleaf template
    }
    
    @PostMapping("/save")
    public String saveOrder(@Valid @ModelAttribute OrderDTO order,
                           BindingResult result) {
        if (result.hasErrors()) {
            return "order-form";
        }
        orderService.save(order);
        return "redirect:/orders/list";
    }
}
```

## Migration Tools

| Tool | Vendor | Capability |
|------|--------|------------|
| CodeLiberator | FreeSoft | Automated Objectstar→Java |
| DataLiberator | FreeSoft | MetaStor extraction |
| KnowledgeLiberator | FreeSoft | Legacy analysis |
| AveriSource | AveriSource | Multi-language migration |
| ModPaaS | Modern Systems | Automated refactoring |

## Migration Checklist

- [ ] Extract complete MetaStor (all tables, rules, screens)
- [ ] Map TDS tables to relational schema
- [ ] Document all parameterized tables
- [ ] Identify LOCAL variable scope chains
- [ ] Map exception handlers to try-catch
- [ ] Convert condition quadrants to decision logic
- [ ] Handle TRANSFERCALL control flow
- [ ] Migrate event rules to entity listeners
- [ ] Convert screens to web UI (MVC)
- [ ] Implement transaction boundaries
- [ ] Create comprehensive test suite
- [ ] Validate business logic preservation
