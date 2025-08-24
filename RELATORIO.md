# Relatório do Laboratório 2 - Chamadas de Sistema

---

## Exercício 1a - Observação printf() vs 1b - write()

### Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: __9__ syscalls
- ex1b_write: __7__ syscalls

**2. Por que há diferença entre os dois métodos? Consulte o docs/printf_vs_write.md**

No write ocorre 7 calls, pois possuem sete writes escritos no exercicio 2, necessitando 7 acessos ao console(syscall).
Ja no print ocorre 9 syscalls, uma vez que possuem nove linhas de texto no arquivo. 

**3. Qual método é mais previsível? Por quê?**
tres

Write pois o aceesso é mais direto e definido, ja o print pode ocorrer um erro e chamar quantidade de cals diferentes devido ao uso do /n e duplicar uma call

---

## Exercício 2 - Leitura de Arquivo

### Resultados da execução:
- File descriptor: 3
- Bytes lidos: 127

### Comando strace:
```bash
strace -e openat,read,close ./ex2_leitura
```

### Análise

**1. Qual file descriptor foi usado? Por que não 0, 1 ou 2?**

```
O file descriptor usado foi o terceiro, uma vez que o 0, 1 e 2 estao reservados para arquivos de entrada padrao como fopen(), para usar fd - um arquivo que e aberto pelo terminal- ele ira ultilizar um numero maior igual a 3
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
Sabemos que o arquivo foi lido corretamente, pois o numero de bytes lidos foi 127 e o maximo é 128, logo o arquivo inteiro foi lido e sobrou um. 
```

**3. O que acontece se esquecer de fechar o arquivo?**

```
Caso esqueca de fechar o arquivo pode ocorrer o vazamento de dados do arquivo, perdendo dados e ocorrendo erros nao esperados.
```

**4. Por que verificar retorno de cada syscall?**

```
E importante verificar o retorno de cada syscall pois facilita na identificação de erros, facilitando nas correçoes fazendo com que o codigo funcione de forma segura e funcione como deve.
```

---

## Exercício 3 - Contador com Loop

### Resultados (BUFFER_SIZE = 64):
- Linhas: __25___ (esperado: 25)
- Caracteres: __1300___
- Chamadas read(): __21___
- Tempo: __0.000095___ segundos

### Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |      82         |  0.000167 |
| 64          |      21         |  0.000095 |
| 256         |       6         |  0.000060 |
| 1024        |       2         |  0.000070 |

### Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
Quanto maior o numero do buffer, menor a quantidade de numero de syscalls, aumentando o numero de buffers mais dados podem ser lidos de uma vez.
```

**2. Como você detecta o fim do arquivo?**

```
O fim do arquivo e detectado quando no loop o read() retorna 0, simbolizando o fim do arquivo. 
```

**3. Todas as chamadas read() retornaram BUFFER_SIZE bytes?**

```
Nao, alguns reads retornam menos que o tamanho do buffer, depende do tipo da chamada, de quantos bytes restam no arquivo, se ocorre um erro ou se a leitura foi finalizada.
```

**4. Qual é a relação entre syscalls e performance?**

```
As syscalls impactam a performance devido ao overhead de troca de contexto entre o espaço de usuário e o kernel! Usar maiores buffers diminui o número de syscalls, diminuindo o overhead e melhorando a eficiência, especialmente em operações de leitura e escrita, ou seja, quanto menos syscalls melhor desempenho.
```

---

## Exercício 4 - Cópia de Arquivo

### Resultados:
- Bytes copiados: __1364___
- Operações: __6___
- Tempo: __0.000274___ segundos
- Throughput: __4861.43___ KB/s

### Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [X] Idênticos [ ] Diferentes

### Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
E importante verificar para garantir que o numero de bytes escritos no arquivo sejam iguais aos lidos, evitando o vazamento de dados.
```

**2. Que flags são essenciais no open() do destino?**

```
As flags essenciais sao O_WRONLY, O_CREAT e O_TRUNC. Cada uma respectivamente abrindo um arquivo para w, criando um arquivo se nao tiver sido criado e e se um arquivo exite apaga e reescreve. Garantindo que o arquivo esteja preparado corretamente segundo as definicoes do ususario.
```

**3. O número de reads e writes é igual? Por quê?**

```
Sim, pois a cada leitura a uma escrita no arquivo de destino, podendo haver uma divergencia na chamada read().
```

**4. Como você saberia se o disco ficou cheio?**

```
Para ver se o disco esta cheio o wite retornaria menos bytes do que o esperado, para isso a condicao bytes_escritos != bytes_lidos garante a verificacao desse erro.
```

**5. O que acontece se esquecer de fechar os arquivos?**

```
Caso haja o esquecimento de fechar os arquivos ocorre o vazamento de dados e memoria, fazendo com que ocorra diversos erros no sistema.
```

---

## Análise Geral

### Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
As syscais sao transicoes entre a execucao do usuario e o kernel, simbolizando o controle do programa e solicitacoes do usuario pelo kernel, permitindo que os programas interajam entre si.
```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
Eles sao essesnciais visto a sua importancia com os dados de entrada e saida de arquivos em um sistema, funcionando como leitura ou escrita e permitindo acesso controlado e eficiente aos recursos dos arquivos.
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
O tamanho do buffer tem relacao direta com a performance, quanto maior o tamanho do buffer mais syscals sao realizadas de uma vez, porem acabam por consumir muita memoria e aumentando o overhead do sistema. Quanto menor o buffer maior a performane.
```

### Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** __user___ (nome do meu usuario no oracle virtualbox)

**Por que você acha que foi mais rápido?**

```
Ele foi mais rapido pois ele opera diretamente no kernel, otimizando as solicitacoes, caches e syscalls.
```

---

## Entrega

Certifique-se de ter:
- [X] Todos os códigos com TODOs completados
- [X] Traces salvos em `traces/`
- [X] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```
