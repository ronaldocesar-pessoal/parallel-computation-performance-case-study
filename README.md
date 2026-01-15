# Reduzindo drasticamente o tempo de cálculos intensivos com computação paralela em CPUs multi-core

Este repositório é um ensaio técnico (engineering case study) sobre como reduzir drasticamente o tempo de execução de um cálculo intensivo por meio de:

- paralelismo de dados em CPU (multi-core)
- decisões de arquitetura para reduzir gargalos de IO
- organização do processamento para maximizar throughput e previsibilidade

O caso real que inspira este ensaio é um trabalho aplicado a cálculo atuarial de larga escala. O artigo técnico original está referenciado ao final.

## Contexto

Em sistemas de cálculo intensivo, é comum observar estes sintomas:

- tempos de processamento que inviabilizam reprocessamento e simulações
- necessidade de rodar o mesmo modelo para múltiplos cenários
- execução anual ou periódica sob pressão de prazo
- aumento contínuo de base de dados e complexidade do modelo

Em muitos ambientes, mesmo com hardware razoável, a aplicação não aproveita a capacidade disponível por executar de forma predominantemente sequencial e por misturar IO e processamento no caminho crítico.

## O problema (em termos de engenharia)

O problema não era “falta de hardware”.
Era uma combinação de:

1) processamento intensivo com alto custo por registro
2) grande volume de registros
3) gargalos de IO no momento errado (durante o cálculo)
4) baixo aproveitamento de CPU por ausência de paralelismo de dados

A consequência prática: o tempo total de processamento se tornava alto demais para rodar com frequência e para produzir cenários em tempo útil.

## Objetivo

Reduzir o tempo total de execução de ponta a ponta, mantendo acurácia e previsibilidade, por meio de:

- paralelização do processamento onde o problema permitia paralelismo de dados
- refatoração para evitar acessos a disco durante a fase de cálculo
- organização do pipeline (carga -> preparação -> cálculo -> agregações)

## Estratégia técnica adotada

A estratégia foi orientada por duas decisões principais:

### Decisão 1: tirar IO do caminho crítico do cálculo

Ao iniciar a aplicação, os dados necessários para o processamento são carregados e organizados em estruturas em memória (ex.: arrays/listas), de modo que, durante o cálculo, a aplicação não dependa de acessos a disco.

Ideia central:
- disco é ordens de grandeza mais lento do que RAM
- se o cálculo acessa dados repetidamente, cachear/estruturar em memória muda o patamar de performance

### Decisão 2: paralelismo de dados em CPU (multi-core)

Com os dados organizados em memória, o próximo passo é explorar paralelismo de dados:

- dividir a coleção de trabalho em partições
- processar as partições em paralelo utilizando múltiplos threads
- controlar concorrência quando existirem recursos compartilhados

O paralelismo foi aplicado com foco em partes do processamento que eram independentes por registro (ou seja, quando o cálculo por indivíduo não dependia do resultado do indivíduo vizinho).

## Notas sobre paralelismo e concorrência

Paralelismo é ferramenta, não dogma. Alguns cuidados que guiaram o uso:

- paralelizar apenas onde existe independência suficiente
- evitar recursos compartilhados no caminho crítico
- quando necessário, usar agregações seguras (por exemplo, somas locais por partição e redução ao final)
- medir sempre antes e depois (benchmark orienta decisão)

Em muitos cenários, “paralelizar tudo” piora: overhead de agendamento, contenção e cache thrashing.

## Resultados (highlights)

Após refatoração e adoção de paralelismo + organização do pipeline, houve redução expressiva no tempo de execução de ponta a ponta, com destaque para:

- importação/carga de dados reduzida drasticamente
- etapas de cálculo intensivo reduzidas de muitas horas para poucos minutos
- tempo total reduzido de dezenas de horas para dezenas de minutos

Os números completos e detalhados estão no artigo referenciado.

## Por que este estudo continua atual

Mesmo com evolução de hardware e cloud, muitas soluções continuam sofrendo de problemas idênticos:

- aplicações rodando em um único thread por padrão
- pipelines que misturam IO com processamento intensivo
- pouca exploração do paralelismo de dados em CPU (multi-core)
- escala de custo “comprada” com infraestrutura em vez de eficiência de software

Antes de pensar em GPU/CUDA, a regra prática é:
- tirar IO do caminho crítico
- organizar dados para acesso eficiente
- explorar multi-core com paralelismo de dados
- medir, ajustar, e só então avaliar aceleração em GPU se houver justificativa

## Como usar este repositório

Este repositório é um ensaio. Ele foi estruturado para ser útil mesmo sem acesso ao código original do sistema.

Sugestão de leitura:
1) ler este README
2) consultar as seções de decisões e benchmarks (arquivos adicionais, se existentes)
3) ler o artigo original para contexto e métricas completas

## Reference paper

This technical essay is inspired by the following peer-reviewed work (written in Portuguese):

**Redução do tempo de processamento do Cálculo Atuarial das Forças Armadas: uma aplicação da Computação Paralela**  
Carlos Francisco Simões Gomes; Marcos dos Santos; Ernesto Rademaker Martins;  
Thierry Faria da Silva Gregório; **Ronaldo Cesar Evangelista dos Santos**  
2017

The full paper is included in this repository as a PDF for long-term reference.
