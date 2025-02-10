# Metodologia de Leitura Incremental e Sumarização Hierárquica para LLMs

## 1. Objetivo e Contextualização

-   **O que se pretende**: Facilitar a compreensão de documentos extensos (e.g., documentação de APIs) usando Modelos de Linguagem com janela de contexto limitada.
-   **Base conceitual**:
    -   **Sumarização Hierárquica** ([Zhang et al., 2020](https://arxiv.org/abs/1912.08777), [Narayan et al., 2018](https://aclanthology.org/P18-1108/)),
    -   **Leitura Multi-hop** (p.ex. [Yang et al., 2018](https://aclanthology.org/D18-1254/)),
    -   **Interação Conversacional** ([Chen et al., 2017](https://arxiv.org/abs/1704.00051)),
    -   **Teoria do Chunking** (Miller, 1956).

A metodologia propõe a divisão do texto em partes (“**chunks**”), a produção de **sumários intermediários**, seguida de interação sistemática entre o usuário (ou pesquisador) e o modelo. Isso permite **acumular conhecimento** sem exceder os limites de contexto e sem perder a coerência global.

----------

## 2. Fluxo Geral

A metodologia pode ser dividida em **5 Fases**:

1.  **Segmentação e Priorização**
2.  **Leitura Inicial e Sumarização Focada**
3.  **Multi-hop: Perguntas e Próxima Leitura**
4.  **Iteração e Fusão de Sumários**
5.  **Síntese Final**

A seguir, detalhamos cada fase com enfoque “passo-a-passo” para que pesquisadores ou profissionais possam reproduzir a técnica.

----------

## 3. Fase 1: Segmentação e Priorização

1.  **Identifique o Escopo e Objetivo**:
    
    -   Defina claramente o que se deseja aprender ou extrair do documento (e.g., “entender como integrar a API X ao projeto Y”).
    -   Baseie-se nos princípios de **Leitura Objetiva** (do QA ou do Summarization) para alinhar expectativas.
2.  **Divida o Documento em Chunks**:
    
    -   Selecione seções relevantes (páginas, capítulos ou tópicos) que **caibam** no limite de tokens do LLM.
    -   Mantenha cada chunk em torno de um tamanho que seja possível **ler** sem estourar a janela de contexto.
3.  **Defina a Ordem de Leitura** (Priorização):
    
    -   Ordene chunks conforme relevância para o seu objetivo.
    -   Considere **índices** ou **sumários** iniciais para guiar a seleção.

> **Resultado**: Uma lista organizada de chunks, priorizados de acordo com o objetivo.

----------

## 4. Fase 2: Leitura Inicial e Sumarização Focada

1.  **Forneça o Contexto Inicial ao LLM**:
    
    -   Explique o objetivo (p.ex., “Quero aprender a usar endpoints de criação e deleção na API X”).
    -   (Opcional) Inclua eventuais **pré-requisitos** já conhecidos ou resumos curtos de partes anteriores.
2.  **Apresente o 1º Chunk de Documentação**:
    
    -   Cole o texto do primeiro chunk.
    -   Solicite um resumo **focado** no objetivo e peça para destacar **conceitos-chave**.
    -   Exemplo de prompt:
        
        > “Aqui está a primeira parte da doc. Resuma com foco em criação/deleção de registros, cite endpoints importantes e potenciais dependências.”
        
3.  **Valide o Resumo**:
    
    -   Leia a saída do LLM.
    -   Ajuste se perceber omissões ou redundâncias.
    -   Guarde esse resumo como **Sumário Intermediário #1**.

> **Resultado**: Você obtém um sumário inicial que descreve o primeiro chunk, orientado aos seus interesses.

----------

## 5. Fase 3: Multi-hop – Perguntas e Próxima Leitura

Inspirado em **Leitura Multi-hop** (Yang et al., 2018), a ideia aqui é permitir que o LLM **solicite** (ou sugira) partes adicionais para aprofundar o entendimento.

1.  **Peça ao LLM Perguntas ou Pontos Pendentes**:
    
    -   Pergunte: “O que ainda precisamos esclarecer? Em quais seções da documentação devemos mergulhar para completar a informação?”
    -   Exemplo de prompt:
        
        > “Com base no Sumário Intermediário #1, quais tópicos adicionais você recomenda ler para entender completamente a criação/deleção de registros?”
        
2.  **Identifique o Próximo Chunk**:
    
    -   Conforme as recomendações do LLM ou sua própria priorização, selecione o próximo trecho.
    -   Mantenha a “conversa” coerente, informando o LLM de onde vem o texto.
3.  **Integre Sumários Anteriores**:
    
    -   Ao enviar o novo chunk, inclua um **breve recall** do sumário anterior (Sumário Intermediário #1) para não perder contexto.
    -   Exemplo de prompt:
        
        > “Sumário anterior: [trechos-chave]. Agora segue o 2º chunk da documentação... Resuma e conecte com o sumário anterior.”
        

> **Resultado**: O modelo continua construindo uma visão incremental, com perguntas servindo de guia para as próximas leituras (técnica próxima do “multi-hop reasoning”).

----------

## 6. Fase 4: Iteração e Fusão de Sumários

1.  **Repita a Operação para Todos os Chunks Importantes**:
    
    -   Para cada novo bloco de texto, gere um **Sumário Intermediário**.
    -   Relacione sempre com os sumários anteriores para consolidar conhecimento.
2.  **Armazene Sumários Intermediários de Forma Sistemática**:
    
    -   Considere numerá-los (Sumário #1, #2 etc.) e guardar cada um em um lugar seguro (seja num documento à parte ou numa base de notas).
3.  **Fusão Parcial de Sumários** (Opcional):
    
    -   A cada certo número de chunks, peça ao LLM um **sumário consolidado** de todos os intermediários gerados até então.
    -   Isso cria um “Sumário Maior” que representa blocos de conteúdo já acumulados, possibilitando liberar espaço no contexto.
4.  **Refinamento**:
    
    -   Caso surjam **inconsistências** ou dúvidas, volte a chunks anteriores ou insira novos trechos.
    -   O processo é iterativo: o modelo pode indicar necessidade de detalhes complementares.

> **Resultado**: Ao final dessa fase, você tem uma coleção de sumários intermediários ou parciais que, unidos, formam uma visão abrangente da documentação.

----------

## 7. Fase 5: Síntese Final

1.  **Gere um Sumário Final Abrangente**:
    
    -   Peça ao LLM que consolide os sumários intermediários em um único texto coerente, destacando:
        -   Pontos-chave
        -   Procedimentos de uso da API
        -   Dicas ou best practices
    -   Solicite diferentes níveis de detalhe: um **sumário executivo** e um **sumário técnico**.
2.  **Confirme Alinhamento com o Objetivo Inicial**:
    
    -   Verifique se o sumário atende ao que foi definido no começo (e.g., “Criar, ler e deletar registros na API X”).
    -   Se faltarem detalhes, retorne à Fase 3 e busque mais chunk(s).
3.  **Documente Referências**:
    
    -   Se a documentação original possuir links ou referências externas, inclua-as no sumário.
    -   Assim, quem precisar de mais profundidade sabe onde encontrar a fonte primária.

> **Resultado**: Você obtém um **Sumário Final** que compacta as informações da documentação, sem ter excedido o limite de tokens em nenhum momento.

----------

## 8. Boas Práticas e Observações Finais

-   **Gerenciar Limite de Tokens**:
    -   Use “chunks” pequenos o suficiente. Em casos extremos, faça sub-chunks ou recorra a sumários parciais para não saturar o contexto.
-   **Verificação Empírica**:
    -   Mesmo que a IA apresente resumos e explicações, é fundamental testar o que foi aprendido em ambiente real ou checar a doc oficial (especialmente se a API ou o texto for complexo e sujeito a atualizações).
-   **Validação de Fatos**:
    -   Modelos de linguagem podem gerar respostas imprecisas ou “alucinações”. Sempre confirme informações críticas.
-   **Reutilização de Sumários**:
    -   Guarde sumários intermediários: eles podem ser incorporados em novos projetos ou compartilhados em equipe.

----------

## 9. Referências Indicadas (Exemplos)

-   **Miller, G.A.** (1956). _The magical number seven, plus or minus two: some limits on our capacity for processing information._ Psychological Review.
-   **Zhang, J. et al.** (2020). _PEGASUS: Pre-training with Extracted Gap-sentences for Abstractive Summarization._ ACL.
-   **Narayan, S. et al.** (2018). _Ranking Sentences for Extractive Summarization with Reinforcement Learning._ ACL.
-   **Yang, Z. et al.** (2018). _HotpotQA: A Dataset for Diverse, Explainable Multi-hop Question Answering._ EMNLP.
-   **Chen, D. et al.** (2017). _Reading Wikipedia to Answer Open-Domain Questions._ ACL.

_(Notas: Essas referências são ilustrações; a integração exata varia conforme o objetivo e o domínio.)_
