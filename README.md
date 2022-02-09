# Aula Prática sobre TDD

Este exercício tem por objetivo emular uma sessão de TDD. O código usado se baseia no exemplo do livro de TDD do Kent Beck.

Objetivo: Sistema que realiza de forma transparente operações em valores de duas moedas (USD e CHF) 
TO-DO List:
  - Precisamos poder trabalhar com diferentes moedas
  - Precisamos poder multiplicar uma quantidade (preço da quota) por um número (quantidade de quotas) e receber o resultado dessa operação (e.g., \$5*2 = \$10)

Passos de Aplicação:

1) Teste e código inicial.
```
public void testMultiplication() {
    Dollar five = new Dollar(5);
    five.times(2);
    assertEquals(10, five.amount);
}
```
```
class Dollar {
    int amount = 10;
    Dollar(int amount) {}			
    void times(int multiplier) {}
}
```
2) Refatoramento da classe Dollar.
```
class Dollar {
   int amount;
   Dollar(int amount) {
      this.amount= amount;
   }
   void times(int multiplier) {
      amount= amount * multiplier;
   }
}	
```
3) Teste que identifica o problema relacionado aos efeitos colaterais de operações sequenciais.
```
public void testMultiplication() {
   Dollar five = new Dollar(5);
   Dollar product = five.times(2);
   assertEquals(10, product.amount);
   product = five.times(3);
   assertEquals(new Dollar(15), product);
}
```
4) Ajuste no método times para tratar o problema identificado em 3).
```
class Dollar {
   ...
   Dollar times(int multiplier) {
      return new Dollar(amount * multiplier);
   }
}
```
5) Refatorando testMultiplication.
```
public void testMultiplication() {
   Dollar five = new Dollar(5);
   assertEquals(new Dollar(10), five.times(2));
   assertEquals(new Dollar(15), five.times(3));
}
```
6) Teste que identifica o problema relacionado a comparação de objetos por valor.
```
public void testEquality() {
   assertTrue(new Dollar(5).equals(new Dollar(5)));
   assertFalse(new Dollar(5).equals(new Dollar(6)));
}
``` 
7) Inclusão do método equals para tratar o problema identificado em 6).
```
public boolean equals(Object object)  {
   Dollar dollar = (Dollar) object;
   return amount == dollar.amount;
}
```
8) Inclusão de teste e código referente ao tratamento de operações com francos suíços.
```
public void testFrancMultiplication() {
   Franc five = new Franc(5);
   assertEquals(new Franc(10), five.times(2));
   assertEquals(new Franc(15), five.times(3));
}

class Franc {   
   private int amount;					
   Franc(int amount) {      
      this.amount= amount;
    }					
    Franc times(int multiplier)  {      
       return new Franc(amount * multiplier);					
    }   
    public boolean equals(Object object) {					
       Franc franc = (Franc) object;      
       return amount == franc.amount;					
     }					
}
```
9) Refatoramento para redução da duplicidade de código
```
class Money  {
   protected int amount;
   
   public boolean equals(Object object)  {
      Money money = (Money) object;
      return amount == money.amount;
   }   
}
```
10) Melhoria do teste que verifica a comparação entre objetos
```
public void testEquality() {
   assertTrue(new Dollar(5).equals(new Dollar(5)));
   assertFalse(new Dollar(5).equals(new Dollar(6)));
   assertTrue(new Franc(5).equals(new Franc(5)));
   assertFalse(new Franc(5).equals(new Franc(6)));
   assertFalse(new Franc(5).equals(new Dollar(5)));
}
```
11) Aprimoramento do método equals para tratar o problema identificado em 10)
```
class Money {
   ...
   public boolean equals(Object object) {
      Money money = (Money) object;
      return amount == money.amount && getClass().equals(money.getClass());
   }
}				
```
12) times agora retorna Money.
```
class Dollar {
   ...
   Money times(int multiplier)  {
      return new Dollar(amount * multiplier);
   }								
}    

class Franc {
   ...
   Money times(int multiplier)  {
      return new Franc(amount * multiplier);
   }								
}    
```
13) As classes Dollar e Franc estão se esvaziando e serão removidas. Nesse sentido, introduzimos métodos fábrica em Money.
```
public void testMultiplication() {
   Money five = Money.dollar(5);
   assertEquals(Money.dollar(10), five.times(2));
   assertEquals(Money.dollar(15), five.times(3));
}
public void testEquality() {
   assertTrue(Money.dollar(5).equals(Money.dollar(5)));
   assertFalse(Money.dollar(5).equals(Money.dollar(6)));
   assertTrue(Money.franc(5).equals(Money.franc(5)));
   assertFalse(Money.franc(5).equals(Money.franc(6)));
   assertFalse(Money.franc(5).equals(Money.dollar(5)));
}
public void testFrancMultiplication() {
   Money five = Money.franc(5);
   assertEquals(Money.franc(10), five.times(2));
   assertEquals(Money.franc(15), five.times(3));
}
```
```
abstract class Money {
   ...
   static Dollar dollar(int amount)  {
      return new Dollar(amount);
   }
   
   static Money franc(int amount) {
      return new Franc(amount);
    }
	
   abstract Money times(int multiplier);  
}
```
14) Inclusão do método currency().
```
public void testCurrency() {
   assertEquals("USD", Money.dollar(1).currency());
   assertEquals("CHF", Money.franc(1).currency());
}
```
```
abstract class Money {
   ...
   abstract String currency();
} 

class Franc extends Money {
   ...
   String currency() {
      return "CHF";
   }
} 

class Dollar extends Money {
    ...
    String currency() {
       return "USD";
    }
} 
```
15) Refatoramento: adicionando um atributo currency, em Franc e Dollar.
```
class Franc extends Money {
   private String currency;
	
   Franc(int amount) {
      this.amount = amount;
      currency = "CHF";
   }
   
   String currency() {
      return currency;
   }
}

class Dollar extends Money {
   private String currency;
	
   Dollar(int amount) {
      this.amount = amount;
      currency = "USD";
   }
   
   String currency() {
      return currency;
   }
}
```
16) Refatoramento: subir currency para Money.
```
abstract class Money {
   ...
   protected String currency;
   ...
   String currency() {
      return currency;
   }
} 

class Franc extends Money {	
   Franc(int amount) {
      this.amount = amount;
      currency = "CHF"; 
   }
   ...
}

class Dollar extends Money {
   Dollar(int amount)  {
      this.amount = amount;
      currency = "USD";
   }
   ...
}
```
17) Refactoramento: times agora retorna Money, tanto em Dollar, como em Franc.
```
class Dollar {
   ...
   Money times(int multiplier)  {
      return Money.dollar(amount * multiplier);
   }								
}    

class Franc {
   ...
   Money times(int multiplier)  {
      return Money.franc(amount * multiplier);
   }								
}    
```
18) Refactoramento: construtores ganham um parâmetro currency.
```
abstract class Money {
   ...
   static Money dollar(int amount)  {
      return new Dollar(amount, "USD");
   }
   
   static Money franc(int amount) {
      return new Franc(amount, "CHF");
   }
} 

class Franc extends Money {
	
   Franc(int amount, String currency) {
      this.amount = amount;
      this.currency = currency;
   }
}

class Dollar extends Money {
	
   Dollar(int amount, String currency)  {
      this.amount = amount;
      this.currency = currency;
   }
}
```
19) Refatoramento: subir construtores para Money.
```
abstract class Money {
   private String currency; 

   static Money dollar(int amount)  {
      return new Dollar(amount, "USD");
   }

   static Money franc(int amount) {
      return new Franc(amount, "CHF");
   }

   Money(int amount, String currency) {
      this.amount = amount;
      this.currency = currency;
   }
}

class Franc extends Money {	
   Franc(int amount, String currency) {
      super(amount, currency);
   }
     
   Money times(int multiplier)  {
      return Money.franc(amount * multiplier);
   }
}

class Dollar extends Money {	
   Dollar(int amount, String currency)  {
      super(amount, currency);
   }
	
   Money times(int multiplier)  {
      return Money.dollar(amount * multiplier);
   }
}
```
20) Novo teste para comparar Money e Franc.
```
public void testDifferentClassEquality() {
   assertTrue(new Money(10, "CHF").equals(new Franc(10, "CHF")));
}
```
21) Ajustes necessários: tornar Money uma classe concreta; modificar o equals para que moedas de classes diferente possam ser iguais, para isso basta que amount e currency sejam iguais; subir times, com alguns ajustes, para Money.
```
class Money {
   ...
   static Money dollar(int amount)  {
      return new Dollar(amount, "USD");
   }
	
   static Money franc(int amount) {
      return new Franc(amount, "CHF");
   }
    
   Money(int amount, String currency) {
      this.amount = amount;
      this.currency = currency;
   }
	
   public boolean equals(Object object) {
      Money money = (Money) object;
      return amount == money.amount && currency().equals(money.currency());
   }
	
   Money times(int multiplier) {
      return new Money(amount * multiplier, currency);
   }
}
```
22) As classes Dollar e Franc não fazem mais sentido de existir. Podem ser removidas. É necessário ajustar as referências na classe Money
```
class Money {
   static Money dollar(int amount)  {
      return new Money(amount, "USD");
   }

   static Money franc(int amount) {
      return new Money(amount, "CHF");
   }
   ...
}
```
