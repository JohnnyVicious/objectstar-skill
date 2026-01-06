# ObjectStar Migration Guide

Guidance for migrating TIBCO ObjectStar applications to modern platforms such as Java, C#, or REXX.

## 1. Migration Preparation
- Inventory all ObjectStar rules, screens, and tables from the MetaStore
- Classify rules: OTP screen logic vs. batch job logic
- Extract dependencies (CALL, EXECUTE, TRANSFERCALL chains)
- Document exception flows and transaction boundaries

## 2. Target Platform Mapping

### Element Mapping
| ObjectStar | Java Equivalent |
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
| TRANSFERCALL | Controller redirect / state machine |
| Event Rule (V) | Bean Validation / @PrePersist |
| Event Rule (T) | @EntityListeners / Triggers |
| Event Rule (D) | @Formula / Computed property |

### Data Type Mapping
| ObjectStar Semantic | ObjectStar Syntax | Java Type |
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

### Exception Mapping
| ObjectStar Exception | Java Equivalent |
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

## 3. Migration Strategy
- **Phase 1:** Refactor ObjectStar code for clarity and modularity (before porting)
- **Phase 2:** Migrate shared business logic first (pure computation rules)
- **Phase 3:** Migrate batch logic (replace FORALL with DB cursors or streaming APIs)
- **Phase 4:** Reimplement screens in UI layer (web, terminal, etc.)
- **Phase 5:** Map transactions and error handling carefully

## 4. Challenges to Expect
- Typeless variables in ObjectStar require type inference in target language
- Implicit intent list behavior must be modeled explicitly
- Dynamic table references (e.g., `TABLES.NAME = "XYZ"; GET TABLES.NAME`) may need refactoring
- Parameterized tables may require separate schemas or tenant-aware logic

## 5. Java Migration Examples

### Condition Quadrant to Java
```
-- ObjectStar
CUST_TYPE = 'PREMIUM';     | Y N N
ORDER_AMT > 1000;          |   Y N
---------------------------+------
DISCOUNT = 0.20;           | 1
DISCOUNT = 0.10;           |   1
DISCOUNT = 0.05;           |     1
```

```java
// Java - Switch expression (Java 14+)
double discount = switch (true) {
    case custType.equals("PREMIUM") -> 0.20;
    case orderAmt > 1000 -> 0.10;
    default -> 0.05;
};

// Java - Decision table pattern
public double calculateDiscount(String custType, BigDecimal orderAmt) {
    if ("PREMIUM".equals(custType)) return 0.20;
    if (orderAmt.compareTo(BigDecimal.valueOf(1000)) > 0) return 0.10;
    return 0.05;
}
```

### FORALL to Java Stream
```
-- ObjectStar
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

### Ordered Iteration
```
-- ObjectStar
FORALL EMPLOYEES ORDERED DESCENDING SALARY UNTIL GETFAIL:
    CALL PRINT_EMP(EMPLOYEES.EMP#);
END;
```

```java
// Java
employeeRepository.findAllByOrderBySalaryDesc()
    .forEach(emp -> printEmp(emp.getEmpNo()));
```

### Exception Handler to try-catch
```
-- ObjectStar
GET CUSTOMERS WHERE CUST# = INPUT_ID;
CALL PROCESS_CUSTOMER;
ON GETFAIL:
    CALL MSGLOG('Customer not found: ' || INPUT_ID);
    SIGNAL CUSTOMER_NOT_FOUND;
```

```java
// Java
try {
    Customer customer = customerRepository.findById(inputId)
        .orElseThrow(() -> new CustomerNotFoundException(inputId));
    processCustomer(customer);
} catch (CustomerNotFoundException e) {
    logger.warn("Customer not found: {}", inputId);
    throw e;
}
```

### LOCAL Variable Scope (Descendant Visibility)
```
-- ObjectStar - LOCAL visible in descendant rules
PARENT_RULE;
LOCAL SHARED_VAR;
SHARED_VAR = 'set in parent';
CALL CHILD_RULE;

CHILD_RULE;
-- Can read/modify SHARED_VAR without declaration
SHARED_VAR = 'modified in child';
```

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

### Screen to Spring MVC
```
-- ObjectStar
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
        return "order-form";
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

## 6. Tooling Suggestions
- Build a parser or use regex to extract rule names, calls, and table access
- Use code classification tags (e.g., `@batch`, `@screen`, `@utility`) to sort rule types
- Maintain an exception mapping dictionary (ObjectStar → target exceptions)

## 7. Verification
- Define behavioral tests for existing rules before migration
- Use log comparison or output parity (e.g., spool files, reports)
- Ensure transactional semantics (commit/rollback) are preserved

## 8. Recommended Refactorings Before Migration
- Eliminate catch-all `ON ERROR` blocks
- Break up large monolithic rules
- Replace screen navigation logic (`DISPLAY`, `TRANSFERCALL`) with explicit state transitions
- Remove or isolate direct screen references in shared logic rules
- Externalize hardcoded values to config tables or parameters

