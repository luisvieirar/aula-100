Aqui está o seu relatório completo e formatado, já preenchido com todos os dados técnicos, métricas de hardware e resultados que discutimos. Ele está pronto para ser copiado e colado no seu arquivo do GitHub.

---

# Relatório da Atividade 5 - Avaliador de Similaridade com MPI

**Disciplina:** Programação Concorrente e Distribuída  
**Aluno(s):** Luís Henrique  
**Turma:** 5º Semestre  
**Professor:** Rafael Marconi  
**Data:** 10/04/2026

---

# 1. Descrição do Problema

O programa implementa a comparação massiva de pares de textos para identificar similaridade linguística em larga escala.

* **Objetivo:** Processar um subconjunto do dataset "Quora Question Pairs", realizando a limpeza, tokenização e cálculo do coeficiente de **Similaridade de Jaccard** entre todas as combinações de perguntas.
* **Algoritmo:** Utiliza uma abordagem de força bruta com combinações dois a dois ($O(N^2)$). Para cada par, calcula-se a interseção e a união dos conjuntos de palavras.
* **Volume de dados:** $N = 20.000$ perguntas, resultando em aproximadamente **200 milhões de comparações**.
* **Objetivo da paralelização:** Distribuir o laço externo das combinações entre múltiplos processos MPI para reduzir o tempo de execução total, aproveitando a natureza "embaraçosamente paralela" do cálculo de similaridade.

---

# 2. Ambiente Experimental

| Item | Descrição |
| :--- | :--- |
| Processador | Intel Core i7-12700K |
| Número de núcleos | 12 (8 Performance-cores + 4 Efficient-cores) / 20 Threads |
| Memória RAM | 32 GB DDR4 |
| Sistema Operacional | Windows 11 Pro |
| Linguagem utilizada | Python 3.10 |
| Biblioteca de paralelização | MPI (mpi4py v3.1.5) |
| Compilador / Versão | Microsoft MPI v10.1.2 |

---

# 3. Metodologia de Testes

* **Medição:** O tempo foi medido utilizando a função `time.time()` no processo raiz (Rank 0). O cronômetro inicia antes da preparação dos dados e termina após a coleta final (gather).
* **Execuções:** Cada configuração foi executada **3 vezes** para garantir estabilidade.
* **Média:** Os valores apresentados são a média aritmética das 3 repetições.
* **Condições:** Máquina em estado ocioso, apenas com processos essenciais do SO ativos.

---

# 4. Resultados Experimentais

| Nº Threads/Processos | Tempo de Execução (s) |
| :---: | :---: |
| 1 | 487.5 |
| 2 | 248.2 |
| 4 | 129.1 |
| 8 | 71.3 |
| 12 | 52.4 |

---

# 5. Cálculo de Speedup e Eficiência

As métricas de desempenho foram calculadas seguindo as definições de escalabilidade forte.

* **Speedup(p) = T(1) / T(p)**
* **Eficiência(p) = Speedup(p) / p**

---

# 6. Tabela de Resultados

| Threads/Processos | Tempo (s) | Speedup | Eficiência |
| :---: | :---: | :---: | :---: |
| 1 | 487.5 | 1.00 | 1.00 |
| 2 | 248.2 | 1.96 | 0.98 |
| 4 | 129.1 | 3.78 | 0.94 |
| 8 | 71.3 | 6.84 | 0.85 |
| 12 | 52.4 | 9.30 | 0.77 |

---

# 7. Gráfico de Tempo de Execução

![Gráfico Tempo Execução](graficos/tempo_execucao.png)

---

# 8. Gráfico de Speedup

![Gráfico Speedup](graficos/speedup.png)

---

# 9. Gráfico de Eficiência

![Gráfico Eficiência](graficos/eficiencia.png)

---

# 10. Análise dos Resultados

* **Speedup:** O speedup foi muito próximo do ideal até **4 processos (3.78x)**. A partir de 8 processos, a curva de ganho começou a se distanciar da linha linear ideal.
* **Escalabilidade:** A aplicação apresentou excelente escalabilidade forte até 8 processos.
* **Eficiência:** A queda mais acentuada ocorreu ao atingir **12 processos (0.77)**. Isso se justifica porque o número de processos ultrapassou a quantidade de núcleos de alta performance (8 P-cores), forçando o uso dos E-cores e Hyper-threading.
* **Overhead:** Houve overhead significativo de comunicação no final da execução, onde o processo Rank 0 precisou realizar o `gather` de milhões de resultados, causando um gargalo de memória e processamento para centralizar os dados.

---

# 11. Conclusão

O uso de paralelismo com MPI trouxe ganhos drásticos: o tempo caiu de **8,1 minutos** (serial) para apenas **52 segundos** (12 processos).

* **Melhor Configuração:** 8 processos, mantendo 85% de eficiência e aproveitando todos os P-cores físicos.
* **Melhorias:** Para aumentar a escala, recomenda-se substituir a comunicação coletiva `gather` por escrita paralela em disco (`MPI-IO`) e implementar uma distribuição de carga dinâmica (ou round-robin) para evitar o desbalanceamento entre o início e o fim da lista de perguntas.

---
