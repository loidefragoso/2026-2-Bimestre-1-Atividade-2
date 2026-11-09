# Relatório de implementação de linhas de execução em Scala

## Introdução

Este relato faz parte do processo avaliativo da disciplina de Sistemas Operacionais do curso superior em Análise e Desenvolvimento de Sistemas, ofertado pela Diretoria Acadêmica de Gestão e Tecnologia da Informação do Campus Natal-Central do Instituto Federal de Educação, Ciência e Tecnologia do Rio Grande do Norte.

O objetivo principal é apresentar como implementar linhas de execução, também chamadas de *threads*, **na linguagem Scala**. Para isso, foram desenvolvidos exemplos de criação de threads, passagem de valores para suas tarefas e execução de múltiplas threads.

O grupo de trabalho foi formado por: Loíde Barros Fragoso, Angelo Evaristo Porfirio e Caio Arthur Cardozo da Silva.

## Implementando múltiplas linhas de execução em Scala

### Informações gerais sobre Scala

Scala é uma linguagem de programação de propósito geral que combina os paradigmas de orientação a objetos e programação funcional. Ela permite trabalhar com classes, objetos e funções, sendo utilizada no desenvolvimento de diferentes tipos de aplicações.

A linguagem está disponível em seu site oficial, [scala-lang.org](https://www.scala-lang.org/), e pode ser utilizada em sistemas como Windows, Linux e macOS. Neste trabalho, foi utilizada a versão **Scala 2.13.12**, executada sobre a Máquina Virtual Java (JVM).

**Todos os códigos da atividade foram escritos em Scala.** O Docker foi utilizado para preparar o ambiente de execução, enquanto o SBT (Scala Build Tool) foi responsável pela compilação e execução do projeto.

O arquivo `build.sbt` recebeu a seguinte configuração:

```scala
scalaVersion := "2.13.12"

name := "Projeto"

version := "1.0"

Compile / mainClass := Some("Main")
```

No arquivo `project/build.properties`, foi definida a versão do SBT:

```properties
sbt.version=1.9.7
```

O ambiente Docker foi configurado por meio do seguinte `Dockerfile`:

```dockerfile
FROM eclipse-temurin:17-jdk

WORKDIR /app

ADD https://repo.maven.apache.org/maven2/org/scala-sbt/sbt-launch/1.9.7/sbt-launch-1.9.7.jar /opt/sbt-launch.jar

COPY . .

RUN java -jar /opt/sbt-launch.jar compile

CMD ["java", "-jar", "/opt/sbt-launch.jar", "run"]
```

Os comandos para construir a imagem e executar o programa são:

```powershell
docker build -t meu-scala .
```

```powershell
docker run --rm -it meu-scala
```

Cada exemplo a seguir corresponde a uma versão do arquivo `src/main/scala/Main.scala`. Após alterar esse arquivo, deve-se reconstruir a imagem para executar a versão atualizada.

### Criando linhas de execução

#### Criação de uma thread em Scala

Em Scala, uma thread pode ser criada utilizando `new Thread` e fornecendo uma tarefa por meio da interface `Runnable`. O método `run()` contém o código que será executado pela nova linha de execução.

```scala
object Main {
  def minhaFuncao(): Unit = {
    println("Thread iniciada!")
    Thread.sleep(2000)
    println("Thread finalizada!")
  }

  def main(args: Array[String]): Unit = {
    val thread = new Thread(new Runnable {
      override def run(): Unit = {
        minhaFuncao()
      }
    })

    thread.start()
    thread.join()

    println("Programa principal finalizado!")
  }
}
```

Nesse código Scala:

- `object Main` define o objeto que contém o programa;
- `def minhaFuncao(): Unit` define uma função que não retorna um valor útil;
- `new Thread` cria uma nova thread;
- `run()` define a tarefa executada pela thread;
- `start()` inicia a nova linha de execução;
- `Thread.sleep(2000)` pausa a thread atual por aproximadamente dois segundos;
- `join()` faz a thread principal aguardar a conclusão da thread criada.

A saída esperada é:

```text
Thread iniciada!
Thread finalizada!
Programa principal finalizado!
```

O programa possui a thread principal e uma thread adicional. Como a principal chama `join()`, a mensagem de encerramento somente aparece após a conclusão da tarefa.

### Passando valores para linhas de execução

Em Scala, podemos definir uma função com parâmetros e chamá-la dentro do método `run()`, informando os valores que serão utilizados pela thread.

No exemplo abaixo, a função `saudar` recebe um nome e a quantidade de mensagens que devem ser exibidas.

```scala
object Main {
  def saudar(nome: String, vezes: Int): Unit = {
    for (i <- 1 to vezes) {
      println(s"Olá, $nome! (mensagem $i)")
    }
  }

  def main(args: Array[String]): Unit = {
    val thread = new Thread(new Runnable {
      override def run(): Unit = {
        saudar("Maria", 3)
      }
    })

    thread.start()
    thread.join()
  }
}
```

Os parâmetros possuem tipos explícitos: `nome` é do tipo `String`, e `vezes` é do tipo `Int`. Na chamada `saudar("Maria", 3)`, esses parâmetros recebem os valores `"Maria"` e `3`.

O laço `for (i <- 1 to vezes)` percorre os números de 1 até a quantidade informada. Já a expressão `s"Olá, $nome! (mensagem $i)"` utiliza a interpolação de strings do Scala para inserir os valores das variáveis na mensagem.

A saída esperada é:

```text
Olá, Maria! (mensagem 1)
Olá, Maria! (mensagem 2)
Olá, Maria! (mensagem 3)
```

### Múltiplas linhas de execução

Para implementar múltiplas threads em Scala, foi criada uma coleção com cinco objetos `Thread`. Cada thread executa a função `trabalhador`, recebendo um número de identificação entre 0 e 4.

```scala
object Main {
  def trabalhador(numero: Int): Unit = {
    println(s"Trabalhador $numero começou")
    Thread.sleep(2000)
    println(s"Trabalhador $numero terminou")
  }

  def main(args: Array[String]): Unit = {
    val threads = (0 until 5).map { i =>
      new Thread(new Runnable {
        override def run(): Unit = {
          trabalhador(i)
        }
      })
    }

    threads.foreach(_.start())

    threads.foreach(_.join())

    println("Todos os trabalhadores terminaram!")
  }
}
```

A expressão `0 until 5` gera os números de 0 a 4. O método `map`, utilizado nas coleções do Scala, transforma cada número em um objeto `Thread`, formando uma coleção com cinco threads.

A instrução:

```scala
threads.foreach(_.start())
```

percorre a coleção e inicia todas as threads. Ela equivale a:

```scala
threads.foreach(thread => thread.start())
```

Depois, a instrução:

```scala
threads.foreach(_.join())
```

faz a thread principal aguardar a conclusão de cada uma das threads criadas. Todas são iniciadas antes dessa espera.

Uma saída possível é:

```text
Trabalhador 0 começou
Trabalhador 2 começou
Trabalhador 1 começou
Trabalhador 3 começou
Trabalhador 4 começou
Trabalhador 2 terminou
Trabalhador 0 terminou
Trabalhador 3 terminou
Trabalhador 1 terminou
Trabalhador 4 terminou
Todos os trabalhadores terminaram!
```

A ordem das mensagens pode variar entre as execuções, pois depende do escalonamento das threads pelo sistema operacional. A mensagem final, entretanto, somente aparece quando todas terminam.

Esse exemplo demonstra a execução concorrente de tarefas em Scala. A execução em paralelo, com tarefas sendo processadas simultaneamente, depende dos recursos disponíveis e do escalonamento do sistema operacional.

## Considerações finais

A atividade apresentou a implementação de linhas de execução **na linguagem Scala**, abordando a criação de uma thread, a passagem de valores para sua tarefa e a execução concorrente de múltiplas threads.

Nos exemplos, foram utilizados recursos da linguagem Scala, como funções declaradas com `def`, objetos definidos com `object`, interpolação de strings e operações sobre coleções com `map` e `foreach`. Para controlar as threads, foram empregados a classe `Thread` e os métodos `start()` e `join()`, acessíveis pela integração do Scala com a JVM.

O Docker forneceu o ambiente necessário para compilar e executar os programas. A lógica de criação, execução e espera das threads foi implementada no código Scala.
