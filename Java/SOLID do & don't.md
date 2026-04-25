# SOLID examples: do and don't

## 1. S — Single Responsibility Principle

**Смысл:** у класса должна быть одна понятная задача.

**Do:**

- `UserRepository` хранит пользователя.
- `UserMailer` отправляет письма.
- `UserValidator` проверяет данные.

**Don't:**

- Один `UserService` и сохраняет, и валидирует, и шлёт email, и пишет логи.

**Почему это важно:** если у класса одна задача, его проще менять и тестировать.

**Простой пример:** если сломалась отправка письма, ты чинишь только mailer, а не весь user-код.

## 2. O — Open/Closed Principle

**Смысл:** код должен быть открыт для расширения, но закрыт для правок в старом месте.

**Do:**

- Добавляй новые типы через новые классы или стратегии.
- Используй интерфейс и разные реализации.

**Don't:**

- Каждый новый случай добавлять в большой `switch` или кучу `if`.

**Почему это важно:** меньше шансов сломать старый код, когда добавляешь новую фичу.

**Простой пример:** вместо `if type == "card" / if type == "cash"` — сделай `PaymentMethod` и отдельные классы.

## 3. L — Liskov Substitution Principle

**Смысл:** любой наследник должен нормально заменять родителя.

**Do:**

- Подкласс ведёт себя так, как ожидают от базового класса.
- Если метод есть в родителе, он не должен неожиданно ломаться в наследнике.

**Don't:**

- Делать класс-наследник, который кидает `UnsupportedOperationException` там, где родитель обещал работу.

**Почему это важно:** если подмена класса ломает программу, наследование сделано неправильно.

**Простой пример:** если есть `Bird`, не надо делать `Penguin extends Bird` и потом удивляться, что он не летает.

## 4. I — Interface Segregation Principle

**Смысл:** лучше несколько маленьких интерфейсов, чем один огромный.

**Do:**

- Разделяй интерфейсы по задачам.
- Класс реализует только то, что реально использует.

**Don't:**

- Делать один интерфейс на 10 методов, где половина классов использует 2-3.

**Почему это важно:** меньше лишнего кода и меньше пустых методов-заглушек.

**Простой пример:** не заставляй принтер уметь сканировать, если он только печатает.

## 5. D — Dependency Inversion Principle

**Смысл:** зависеть надо от абстракций, а не от конкретных классов.

**Do:**

- Работай через интерфейсы.
- Передавай зависимости снаружи, а не создавай их прямо внутри.

**Don't:**

- Жёстко создавать `new MySqlUserRepository()` прямо в бизнес-логике.

**Почему это важно:** код легче менять, подменять и тестировать.

**Простой пример:** сегодня база MySQL, завтра PostgreSQL — бизнес-логика не должна переписываться.

## Быстрые правила

- Один класс — одна задача.
- Не лепи `switch` на всё подряд.
- Не заставляй наследников ломать ожидания.
- Не делай огромные интерфейсы.
- Не привязывай бизнес-логику к конкретным реализациям.

## Короткая памятка

| Принцип | Do                                | Don't                              |
|---------|-----------------------------------|------------------------------------|
| SRP     | Один класс — одна задача          | Один класс делает всё              |
| OCP     | Добавляй новое через расширение   | Лезь в старый код ради каждой фичи |
| LSP     | Наследник ведёт себя как родитель | Подкласс ломает ожидания           |
| ISP     | Много маленьких интерфейсов       | Один жирный интерфейс              |
| DIP     | Зависимость от интерфейсов        | Зависимость от конкретных классов  |

## Как запомнить

- **S** — разделяй задачи.
- **O** — добавляй новое без переписывания старого.
- **L** — наследник не должен сюрпризить.
- **I** — не заставляй класс делать лишнее.
- **D** — завись от интерфейсов, а не от конкретики.

## Code examples

### 1. SRP

**Плохо:** один класс делает всё.

```java
class UserService {
    void saveUser(User user) {
    }

    void sendWelcomeEmail(User user) {
    }

    void validateUser(User user) {
    }
}
```

**Хорошо:** у каждого класса своя задача.

```java
class UserRepository {
    void save(User user) {
    }
}

class UserValidator {
    void validate(User user) {
    }
}

class UserMailer {
    void sendWelcomeEmail(User user) {
    }
}
```

### 2. OCP

**Плохо:** каждый новый способ оплаты требует правки старого `switch`.

```java
class PaymentService {
    double pay(String type, double amount) {
        if (type.equals("card")) return amount * 0.9;
        if (type.equals("cash")) return amount;
        return amount;
    }
}
```

**Хорошо:** добавляй новый класс, старый код не трогай.

```java
interface Discount {
    double apply(double amount);
}

class CardDiscount implements Discount {
    public double apply(double amount) {
        return amount * 0.9;
    }
}

class CashDiscount implements Discount {
    public double apply(double amount) {
        return amount;
    }
}
```

### 3. LSP

**Плохо:** наследник ломает ожидаемое поведение.

```java
class Bird {
    void fly() {
    }
}

class Penguin extends Bird {
    void fly() {
        throw new UnsupportedOperationException();
    }
}
```

**Хорошо:** не заставляй класс делать то, что ему не подходит.

```java
interface Bird {
}

interface FlyingBird extends Bird {
    void fly();
}

class Sparrow implements FlyingBird {
    public void fly() {
    }
}

class Penguin implements Bird {
}
```

### 4. ISP

**Плохо:** большой интерфейс с лишними методами.

```java
interface Worker {
    void work();

    void eat();

    void sleep();
}
```

**Хорошо:** маленькие интерфейсы по делу.

```java
interface Workable {
    void work();
}

interface Eatable {
    void eat();
}
```

### 5. DIP

**Плохо**
Бизнес-логика жёстко зависит от конкретной реализации:

```java
class NotificationService {
    private final EmailSender emailSender = new EmailSender();

    void notifyUser(String message) {
        emailSender.send(message);
    }
}
```

Проблема в том, что `NotificationService` привязан именно к `EmailSender`. Если завтра захочешь отправлять не email, а SMS или Telegram,
придётся лезть в этот класс и переписывать его.

**Хорошо**
Зависимость идёт через абстракцию:

```java
interface MessageSender {
    void send(String message);
}

class EmailSender implements MessageSender {
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}

class SmsSender implements MessageSender {
    public void send(String message) {
        System.out.println("SMS: " + message);
    }
}

class NotificationService {
    private final MessageSender sender;

    NotificationService(MessageSender sender) {
        this.sender = sender;
    }

    void notifyUser(String message) {
        sender.send(message);
    }
}
```

**Почему это лучше**

- `NotificationService` не знает, как именно отправляется сообщение.
- Можно легко подменить email на SMS или Telegram.
- Код проще тестировать: можно подставить фейковый sender.

