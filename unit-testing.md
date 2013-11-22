# Модульное тестирование

На данный момент можно выделить два наиболее распространненых подхода к написанию модульных тестов.
Речь идет о классическом и поведенческом тестировании.
Эти подходы являются взаимодополняющими, и их комбинирование позволяет повысить эффективность модульного тестирования в целом.

## Введение

Поведенческое тестирование отличается от классического в первую очередь тем, что основывается на допущениях об ожидаемом поведении тестируемого объекта.
По сути поведенческие тесты направлены на проверку этих допущений.
На следующих примерах будет показано, каким образом выполняется построение таких тестов и какие преимущества они дают.

## Тестирование метода двоичного поиска

Вначале рассмотрим пример классического и поведенческого тестов для метода binarySearch класса Collections.

#### Классический тест

```java
public class BinarySearchClassicalTest {

    @Test
    public void testEmpty() {
        // setup
        List<Integer> list = Collections.emptyList();

        // exercise
        int pos = binarySearch(list, 4);

        // verify
        assertEquals(-1, pos);
    }

    @Test
    public void testRegular() {
        // setup
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        // exercise
        int pos = binarySearch(list, 4);

        // verify
        assertEquals(3, pos);
    }

}
```

Каждый модульный тест включает четыре стадии: подготовку (setup), выполнение (exercise), проверку (verify) и завершение (teardown).

В классических тестах на стадии подготовки создаются объекты, необходимые для тестирования.
В данном случае это вспомогательный объект - список целых чисел.
После этого, на стадии выполнения вызывается тестируемый метод.
Наконец, на стадии проверки полученный результат сравнивается с ожидаемым значением.
Для выполнения проверки в классических тестах зачастую используется механизм утверждений (assertions).

Для классических тестов характерно что объект тестирования рассматривается как черный ящик (black box).
То есть проверяется только результат, возвращенный тестируемым методом, без учета того, каким образом он был получен.

#### Поведенческий тест

```java
public class BinarySearchBehavioralTest {

    @Test
    public void testEmpty() {
        // setup
        List<Integer> mockList = spy(new ArrayList<Integer>());

        // exercise
        binarySearch(mockList, 4);

        // verify
        verify(mockList).size();
        verifyNoMoreInteractions(mockList);
    }

    @Test
    public void testRegular() {
        // setup
        List<Integer> list = new ArrayList<Integer>();
        list.addAll(Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10));
        List<Integer> mockList = spy(list);

        // exercise
        binarySearch(mockList, 4);

        // verify
        verify(mockList).size();
        verify(mockList).get(4);
        verify(mockList).get(1);
        verify(mockList).get(2);
        verify(mockList).get(3);
        verifyNoMoreInteractions(mockList);
    }

}
```

В поведенческих тестах на стадии подготовки также создаются необходимые объекты.
Однако, это не реальные объекты, а _тестовые объекты_ (test doubles), которые имитируют их поведение.
В данном случае создается _наблюдаемый объект_, который ведет себя как реальный, но позволяет проверить вызовы его методов.
На стадии выполнения наблюдаемый объект передается в тестируемый метод вместо реального объекта.

Рассмотрим проверку поведения в тесте testRegular.
Вначале, с помощью метода verify, проверяется, что метод List.size был вызван ровно один раз для объекта mockList.
Затем проверяется, что метод List.get для этого объекта был вызван четыре раза с аргументами 4, 1, 2 и 3.
Однако при этом не проверяется очередность вызовов.
Наконец, с помощью метода verifyNoMoreInteractions, проверяется то, что никаких других обращений к объекту mockList не было.

То есть, построение поведенческого теста требует допущений о реализации объекта тестирования.

## Виды тестовых объектов и их использование

Можно выделить четыре вида тестовых объектов:

1. dummy - передаются в качестве аргументов методов, но никогда не используются.
2. stub - возвращают предопределенные результаты, также могут выполнять протоколирование вызовов.
3. fake - имеют упрощенную реализацию, предназначенную только для тестирования.
4. mock - имеют предопределенное поведение, необходимое для конкретного тестового сценария.

Использование dummy, stub и fake объектов характерно для классических тестов.
В поведенческих тестах используются только mock-объекты.

Рассмотрим пример использования stub и mock объектов в классическом и поведенческом тестах.

#### Классический тест

```java
public class ApproveOrderClassicalTest {

    @Test
    public void testApproved() {
        // setup
        Order order = new Order();
        order.setCardNumber("4111111111111111");
        order.setTotal(500.00);

        PaymentServiceStub paymentService = new PaymentServiceStub();
        order.setPaymentService(paymentService);

        // exercise
        order.approve();

        // verify
        assertTrue(order.isApproved());
        assertEquals(1, paymentService.getTotalApproved());
    }

}
```

В этом тесте объектом тестирования является метод Order.approve.
Однако для выполнения этого метода необходим объект класса PaymentService.
Поэтому класс PaymentService является зависмым классом (collaborator или dependent-on component) для класса Order.

В качестве объекта класс PaymentService используется stub-объект.
Этот stub-объект подсчитывает количество утвержденных выплат, что позволяет проверять его состояние в тесте.

```java
public interface PaymentService {

    boolean approve(String cardNumber, double amount);

}

public class PaymentServiceStub implements PaymentService {

    private int totalApproved;

    @Override
    public boolean approve(String cardNumber, double amount) {
        if (!"4111111111111111".equals(cardNumber)) {
            return false;
        }

        ++totalApproved;
        return true;
    }

    public int getTotalApproved() {
        return totalApproved;
    }

}
```

Данный тест иллюстрирует, что классическое тестирование основывается на проверке состояния объектов (state verification).
То есть, на стадии проверки мы убеждаемся, что состояние всех объектов соответствует ожиданиям.
При этом не учитывается, каким образом это состояние было достигнуто, так как каждый объект рассматривается как черный ящик.

#### Поведенческий тест

```java
public class ApproveOrderBehavioralTest {

    @Test
    public void testApproved() {
        // setup data
        Order order = new Order();
        final String code = "4111111111111111";
        order.setCardNumber(code);
        final double amount = 500.00;
        order.setTotal(amount);

        PaymentService paymentService = mock(PaymentService.class);
        order.setPaymentService(paymentService);

        // setup behavior
        doReturn(false).when(paymentService).approve(anyString(), anyDouble());
        doReturn(true).when(paymentService).approve(eq(code), anyDouble());

        // exercise
        order.approve();

        // verify state
        assertTrue(order.isApproved());

        // verify behavior
        verify(paymentService).approve(code, amount);
        verifyNoMoreInteractions(paymentService);
    }

}
```

В поведенческом тесте стадия подготовки разделена на две части.
В первой части выполняется подготовка данных, аналогично классическому тесту.
Отличием является то, что в качестве объекта класса PaymentService используется mock-объект.
Вторая часть характерна только для поведенческих тестов, поскольку в ней описывается ожидаемое поведение mock-объектов.
Вначале определяется, что при вызове метода PaymentService.approve с любыми аргументами должно быть возвращено значение false.
После этого уточняется, что при вызове этого метода для определенной кредитной карты должно возвращаться значение true.

После выполнения метода Order.approve проверяется, что метод PaymentService.approve был вызван один раз с определенными аргументами.
Также проверяется, что объект класса PaymentService больше никак не использовался.
То есть поведенческий тест базируется на проверке поведения (behavior verification).
Тем не менее, проверка поведения применяется только к mock-объектам.
Для реальных объектов, например, для тестируемого объекта, может применяться проверка состояния.

## Выводы

Классические и поведенческие тесты позволяют выполнять тестирование исходя из разных предпосылок.
В классическом тесте тестируемый объект рассматривается как черный ящик, а потому проверяется его состояние или результаты выполнения его методов.
В свою очередь, для написания поведенческого тесты необходимо наличие допущений о реализации тестируемого объекта.
Этот вид тестов позволяет проверить правильность взаимодействия объекта с его окружением.
