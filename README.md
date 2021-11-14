# 1. Visão Geral
Neste artigo, falaremos sobre um recurso central da linguagem Java - as anotações padrão disponíveis no JDK.

# 2. O que é uma anotação
Simplificando, as anotações são tipos Java precedidos por um símbolo “@”.

Java teve anotações desde o lançamento 1.5. Desde então, eles moldaram a maneira como projetamos nossos aplicativos.

Spring e Hibernate são ótimos exemplos de frameworks que dependem fortemente de anotações para habilitar várias técnicas de design.

Basicamente, uma anotação atribui metadados extras ao código-fonte ao qual está vinculada. Ao adicionar uma anotação a um método, interface, classe ou campo, podemos:

1 - Informar ao compilador sobre avisos e erros;
2 - Manipular o código fonte em tempo de compilação;
3 - Modifique ou examine o comportamento em tempo de execução.

# 3. Anotações integradas em Java
Agora que revisamos o básico, vamos dar uma olhada em algumas anotações que vêm com o Java principal. Primeiro, existem vários que informam a compilação:

1 - @Override;
2 - @SuppressWarnings;
3 - @Deprecated;
4 - @SafeVarargs;
5 - @FunctionalInterface;
6 - @Native.

Essas anotações geram ou suprimem avisos e erros do compilador. Aplicá-los consistentemente costuma ser uma boa prática, pois adicioná-los pode evitar erros futuros do programador.

A anotação @Override é usada para indicar que um método sobrescreve ou substitui o comportamento de um método herdado.

@SuppressWarnings indica que desejamos ignorar certos avisos de uma parte do código. A anotação @SafeVarargs também atua em um tipo de aviso relacionado ao uso de varargs.

A anotação @Deprecated pode ser usada para marcar uma API como não destinada a ser usada mais. Além disso, essa anotação foi adaptada no Java 9 para representar mais informações sobre a reprovação.

Para tudo isso, você pode encontrar informações mais detalhadas nos artigos vinculados.

### 3.1. @FunctionalInterface
Java 8 nos permite escrever código de uma maneira mais funcional.

Interfaces de método abstrato único são uma grande parte disso. Se pretendemos que uma interface SAM seja usada por lambdas, podemos opcionalmente marcá-la como tal com @FunctionalInterface:

```
@FunctionalInterface
public interface Adder {
    int add(int a, int b);
}
```

Como @Override com métodos, @FunctionalInterface declara nossas intenções com o Adder.

Agora, quer usemos @FunctionalInterface ou não, ainda podemos usar o Adder da mesma maneira:

```
Adder adder = (a,b) -> a + b;
int result = adder.add(4,5);
```

Mas, se adicionarmos um segundo método ao Adder, o compilador reclamará:

```
@FunctionalInterface
public interface Adder { 
    // compiler complains that the interface is not a SAM
    
    int add(int a, int b);
    int div(int a, int b);
}
```

Agora, isso teria sido compilado sem a anotação @FunctionalInterface. Então, o que isso nos dá?

Como @Override, essa anotação nos protege contra erros futuros do programador. Embora seja legal ter mais de um método em uma interface, não é quando essa interface está sendo usada como um destino lambda. Sem essa anotação, o compilador iria quebrar nas dezenas de lugares onde o Adder foi usado como lambda. Agora, ele apenas quebra no próprio Adder.

### 3.2. @Nativo
A partir do Java 8, há uma nova anotação no pacote java.lang.annotation chamada Native. A anotação @Native é aplicável apenas a campos. Indica que o campo anotado é uma constante que pode ser referenciada a partir do código nativo. Por exemplo, é assim que ele é usado na classe Integer:

```
public final class Integer {
    @Native public static final int MIN_VALUE = 0x80000000;
    // omitted
}
```

Essa anotação também pode servir como uma dica para as ferramentas gerarem alguns arquivos de cabeçalho auxiliares.

# 4. Meta-anotações
Em seguida, meta-anotações são anotações que podem ser aplicadas a outras anotações.

Por exemplo, essas meta-anotações são usadas para configuração de anotação:

1 - @Target;
2 - @Retention;
3 - @Inherited;
4 - @Documented;
5 - @Repetível.

### 4.1. @Alvo
O escopo das anotações pode variar de acordo com os requisitos. Enquanto uma anotação é usada apenas com métodos, outra anotação pode ser consumida com declarações de construtor e campo.

Para determinar os elementos de destino de uma anotação personalizada, precisamos rotulá-la com uma anotação @Target.

@Target pode trabalhar com 12 tipos de elementos diferentes. Se olharmos o código-fonte de @SafeVarargs, então podemos ver que ele deve ser anexado apenas a construtores ou métodos:

```
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD})
public @interface SafeVarargs {
}
```

### 4.2. @Retenção
Algumas anotações devem ser usadas como dicas para o compilador, enquanto outras são usadas em tempo de execução.

Usamos a anotação @Retention para dizer onde, no ciclo de vida do nosso programa, nossa anotação se aplica.

Para fazer isso, precisamos configurar @Retention com uma das três políticas de retenção:

1 - RetentionPolicy.SOURCE - visível nem pelo compilador nem pelo tempo de execução;
2 - RetentionPolicy.CLASS - visível pelo compilador;
3 - RetentionPolicy.RUNTIME - visível pelo compilador e pelo tempo de execução.

Se nenhuma anotação @Retention estiver presente na declaração de anotação, a política de retenção será padronizada como RetentionPolicy.CLASS.

Se tivermos uma anotação que deve ser acessível em tempo de execução:

```
@Retention(RetentionPolicy.RUNTIME)
@Target(TYPE)
public @interface RetentionAnnotation {
}
```

Então, se adicionarmos algumas anotações a uma classe:

```
@RetentionAnnotation
@Generated("Available only on source code")
public class AnnotatedClass {
}
```

Agora podemos refletir sobre AnnotatedClass para ver quantas anotações são retidas:

```
@Test
public void whenAnnotationRetentionPolicyRuntime_shouldAccess() {
    AnnotatedClass anAnnotatedClass = new AnnotatedClass();
    Annotation[] annotations = anAnnotatedClass.getClass().getAnnotations();
    assertThat(annotations.length, is(1));
}
```

O valor é 1 porque @RetentionAnnotation tem uma política de retenção de RUNTIME, enquanto @Generated não.

### 4.3. @Inherited
Em algumas situações, podemos precisar de uma subclasse para ter as anotações vinculadas a uma classe pai.

Podemos usar a anotação @Inherited para fazer nossa anotação se propagar de uma classe anotada para suas subclasses.

Se aplicarmos @Inherited à nossa anotação personalizada e, em seguida, aplicá-la a BaseClass:

```
@Inherited
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface InheritedAnnotation {
}

@InheritedAnnotation
public class BaseClass {
}

public class DerivedClass extends BaseClass {
}
```

Então, depois de estender a BaseClass, devemos ver que DerivedClass parece ter a mesma anotação em tempo de execução:

```
@Test
public void whenAnnotationInherited_thenShouldExist() {
    DerivedClass derivedClass = new DerivedClass();
    InheritedAnnotation annotation = derivedClass.getClass()
      .getAnnotation(InheritedAnnotation.class);
 
    assertThat(annotation, instanceOf(InheritedAnnotation.class));
}
```

Sem a anotação @Inherited, o teste acima falharia.

### 4.4. @Documented
Por padrão, Java não documenta o uso de anotações em Javadocs.

Mas, podemos usar a anotação @Documented para alterar o comportamento padrão do Java.

Se criarmos uma anotação personalizada que usa @Documented:

```
@Documented
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ExcelCell {
    int value();
}
```

E aplique-o ao elemento Java apropriado:

```
public class Employee {
    @ExcelCell(0)
    public String name;
}
```

Em seguida, o Javadoc do funcionário revelará o uso da anotação.

### 4.5. @Repetivel
Às vezes, pode ser útil especificar a mesma anotação mais de uma vez em um determinado elemento Java.

Antes do Java 7, tínhamos que agrupar as anotações em uma única anotação de contêiner:

```
@Schedules({
    @Schedule(time = "15:05"),
    @Schedule(time = "23:00")
})
void scheduledAlarm() {
}
```

No entanto, o Java 7 trouxe uma abordagem mais limpa. Com a anotação @Repeatable, podemos tornar uma anotação repetível:

```
@Repeatable(Schedules.class)
public @interface Schedule {
    String time() default "09:00";
}
```

Para usar @Repeatable, precisamos ter uma anotação de contêiner também. Nesse caso, vamos reutilizar @Schedules:

```
public @interface Schedules {
    Schedule[] value();
}
```

Claro, isso se parece muito com o que tínhamos antes do Java 7. Mas, o valor agora é que o wrapper @Schedules não é mais especificado quando precisamos repetir @Schedule:

```
@Schedule
@Schedule(time = "15:05")
@Schedule(time = "23:00")
void scheduledAlarm() {
}
```

Como o Java requer a anotação do wrapper, foi fácil para nós migrar das listas de anotações pré-Java 7 para anotações repetíveis.

# 5. Conclusão
Neste artigo, falamos sobre as anotações integradas de Java com as quais todo desenvolvedor Java deve estar familiarizado.
