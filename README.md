# **Prova da Disciplina de Sistemas Distribuídos**
##### Aluna: Camila Gonçalves Frizzo
##### Professor: Francisco 
O código abaixo apresenta um exemplo clássico de algoritmo combinatório para encontrar as soluções da soma de uma lista de valores. Descrição do Problema: O Computador deve gerar uma lista de Números Naturais aleatórios, de dimensão passada por parâmetro, que representa os índices de medição de chuva. Uma soma de valor '15' foi atribuída para efeito de cálculo. O programa deve retornar as combinações que apresentam a soma da lista igual ao valor definido (no caso 15).

## Código 1

~~~python
%%writefile codigoobi.py

#Importação das bibliotecas

import os

import random

import sys

  

random.seed(1) #Utilizado aqui para efeito de estudo, pois os mesmos valores

#serão sorteados sempre, facilitando a explicação.

  

#Recebendo a lista de valores por parâmetro do sistema

tam_medicao = int(sys.argv[1])

SOMA = 15 #Valor de soma atribuído - Poderia ser qualquer valor.

  

combinacoes = [] #Esta lista guarda todas as combinações de somas possíveis.

  

#Gerando as combinações de números da lista possíveis, dependendo do tamanho

#passado por parâmetro (tam_medicao).

#As combinações de números da lista será de ordem "2^tam_medicao", iniciando

#em: 0000...0 até 1111...1.

print('\nCombinações: ')

for i in range(0, 2**tam_medicao):

x = bin(i)

#print(x)

combinacoes.append(x.split('b')[1].zfill(tam_medicao))

print(combinacoes)

print()

  

#Serão gerados aleatóriamente uma quantidade de números definida por 'tam_medicao'

medicao = ''

for x in range(0,tam_medicao):

medicao += str(random.randint(1,10)) + ' '

medicao = medicao[:len(medicao)-1].split(' ')

print('\nMedição:')

print(medicao)

  

print('\nCombinações que apresentam a soma %d' %SOMA)

total = 0

for l in combinacoes:

soma = 0

fatores = list(l)

for pos in range(0, len(l)):

soma += int(fatores[pos]) * int(medicao[pos])

if soma == SOMA:

total += 1

print(medicao)

print(fatores)

print()

print('Total de combinações com a soma %d: %d' % (SOMA, total))
~~~
#### **Script1**
Executando o código acima passando o parâmetro de 8
~~~python
 !python codigoobi.py 8
~~~

Podemos observar quais as combinações possíveis de 8 números: 00000000 - 11111111. Em seguida, o código mostra a lista de 8 números sorteados aleatoriamente. Assim, não é difícil definir quais as combinações de números que apresentem uma soma igual ao definido no algoritmo(15).

**Verificando o crescimento do custo computacional para resolver o código anterior**
Como o tamanho do problema (a quantidade de números), pode ser passada por parâmetro, podemos monitorar o tempo de execução do código em cada situação, 
 dependendo da quantidade de números da lista.
 
#### **Script2**
~~~python
import time as t

tempo_exec=[]

tempo_teorico=[]

for i in  range(15,  23):

tempo_atual = t.time()

string = 'python codigoobi.py '+str(i)+' >> /dev/null'  #Jogando a saida para null para não poluir o terminal

!$string

tempo_final = t.time()

tempo_final -= tempo_atual

tempo_exec.append((i,tempo_final))

#Comparando os resultados com os valores de f(i) = 2*f(i-1)

tempo_teorico.append((i+1, tempo_final*2))

print(tempo_exec)

print(tempo_teorico)
~~~

Como podemos observar, existe uma grande variação de tempo de execução ao alterar uma pequena quantidade de números da lista: 
  

* 15 números ~= 0.42 segundos

* 22 números ~= 51.61 segundos.  

Outra característica interessante que podemos observar é a aproximação do tempo de execução em um determinado instante ser semelhante ao dobro do tempo no instante anterior:  

$f(x) = 2^t$ 

Onde $t$ é a quantidade de números da lista

### **Plotando o gráfico do Tempo de Execução:**
Como podemos observar no gráfico anterior, o tempo de execução foi próximo de 0 para uma lista de 15 números e rapidamente cresceu para mais que 50 segundos ao chegarmos em 22 números.
~~~python
import matplotlib.pyplot as plt

plt.figure(figsize=(10,7))

plt.plot([tempo_exec[i][0]  for i in  range(len(tempo_exec))],  [tempo_exec[i][1]  for i in  range(len(tempo_exec))], label='Alg.SingleCore')

plt.plot([tempo_teorico[i][0]  for i in  range(len(tempo_teorico)-1)],  [tempo_teorico[i][1]  for i in  range(len(tempo_teorico)-1)], label='Previsto')

plt.title('Tempo de execução')

plt.xlabel('Tamanho da Lista')

plt.ylabel('Segundos')

plt.legend()

plt.grid()

plt.show()
~~~
## **Paralelização do Problema**

Como podemos observar, temos um código com combinações de uma lista de números que são selecionados para obter uma soma. Isto pode ser caracterizado como um problema de combinação de valores e portanto pode ser facilmente paralelizado.

Para tanto, podemos entregar a cada unidade de processamento um intervalo da combinação a ser testada.

Exemplo:

Considerando um problema com a lista de números: [10, 5, 2, 3], onde desejamos obter as combinações de números cuja a soma corresponda à (10).

Neste caso, o computador pode testar todas as combinações:  

* [0, 0, 0, 0] - Não levar nenhum produto até,

* [1, 1, 1, 1] - Levar todos os produtos. 

Facilmente podemos observar que as soluções podem ser encontradas em:  

* [0, 1, 1, 1] e

* [1, 0, 0, 0].  

Nada impede também que os testes sejam divididos em unidades de processamento individuais, pois os resultados dos testes não são interdependentes. Assim a unidade de processamento 1 (P1), poderia realizar os testes entre: 0000 e 0111; enquanto a unidade de processamento 2 (P2), poderia realizar os testes entre: 1000 e 1111; de forma paralela. 

Ao final de todos os testes as unidades de processamento poderiam trocar mensagens afim de informar seus resultados parciais, unificando-os.

### **OpenMPI**
Observando a utilização do OpenMPI em Python, o "Código1" foi modificado, afim de paralelizar as tarefas de testes de combinações da soma da lista de números em dois processos distintos.
Os gráficos do tempo de execução de cada algoritmo (Serial e Paralelo) foram plotados, afim de observar o ganho de processamento.

## Código 2
~~~python
%%writefile codigoobi2.py

from mpi4py import MPI 

import os
import sys
import random  
  

tam_medicoes = int(sys.argv[1]) 

comm = MPI.COMM_WORLD

id = comm.Get_rank()

numerodeprocessos = comm.Get_size()
  

inicio = round(id * ((2**tam_medicoes) / numerodeprocessos))

fim = round(inicio + ((2**tam_medicoes) / numerodeprocessos) -1)
  

print('Processo: %d | inicio: %d - fim:%d' %(id,inicio,fim))
  

soma_usuario = 15

medicao = ''
  

if id == 0:

for x in range(0,tam_medicoes):

medicao += str(random.randint(1,10)) + ' '

medicao = medicao[:len(medicao)-1].split(' ')

print('A medição é:')

print(medicao)

print()

for i in range(1, numerodeprocessos):

comm.send(medicao, dest= i, tag=22)

else:

medicao = comm.recv(source=0, tag=22)
  

combinacoes = []

for i in range(inicio, fim + 1):

x = bin(i)

combinacoes.append(x.split('b')[1].zfill(tam_medicoes))

print('As combinações são:')

print(combinacoes)

print() 
  

total = 0

resultato_fatores = []
  

for l in combinacoes:

soma = 0

fatores = list(l)

for pos in range(0, len(l)):

soma += int(fatores[pos]) * int(medicao[pos])

if soma == soma_usuario:

total += 1

resultato_fatores.append(fatores) 
  

if id != 0:

comm.send(resultato_fatores, 0)

else:

for i in range(0, numerodeprocessos -1):

resultato_fatores.append(comm.recv())  

print('Total de combinações com a soma no processo %d: %d' % (id,total))
~~~
#### **Script3**
Executando o código acima, sendo o tamanho das medições 8 e o valor da soma 15, assim como no teste do "Código 1".
~~~python
 !mpirun --allow-run-as-root -np 2 python codigoobi2.py 8
~~~
#### **Script4**
~~~python
 import time as t

tempo_exec2=[]

tempo_teorico2=[]

for i in  range(15,  23):

tempo_atual = t.time()

string = 'mpirun --allow-run-as-root -np 2 python codigoobi2.py '+str(i)+' >> /dev/null'  #Jogando a saida para null para não poluir o terminal

!$string

tempo_final2 = t.time()

tempo_final2 -= tempo_atual

tempo_exec2.append((i,tempo_final2))

#Comparando os resultados com os valores de f(i) = 2*f(i-1)

tempo_teorico2.append((i+1, tempo_final2*2))

print(tempo_exec2)

print(tempo_teorico2)
~~~
Como podemos observar, a variação de tempo de execução ao alterar uma pequena quantidade de números da lista performou de forma melhor do que no Código 1:  

* 15 números ~= 0.31 segundos
* 22 números ~= 0.32 segundos.

### **Plotando o gráfico do Tempo de Execução:**
Como podemos observar no gráfico anterior, o tempo de execução ficou entre um intervalo de 0 a 1 para uma lista de 15 números e também para uma lista de 22 números.

~~~python
import matplotlib.pyplot as plt

plt.figure(figsize=(10,7))

plt.plot([tempo_exec2[i][0]  for i in  range(len(tempo_exec2))],  [tempo_exec2[i][1]  for i in  range(len(tempo_exec2))], label='Alg.MultiCore')

plt.plot([tempo_teorico2[i][0]  for i in  range(len(tempo_teorico2)-1)],  [tempo_teorico2[i][1]  for i in  range(len(tempo_teorico2)-1)], label='Previsto')

plt.title('Tempo de execução')

plt.xlabel('Tamanho da Lista')

plt.ylabel('Segundos')

plt.legend()

plt.grid()

plt.show()
~~~
### Link para teste do código
Segue um link do Google Colaboratory onde é possível executar os códigos apresentados aqui:
<https://colab.research.google.com/drive/1yOSyG5I2WGv491kRaj4cL4Z5yBoGrr5j?usp=sharing>
