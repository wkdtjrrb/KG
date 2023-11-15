# Knowledge Graph Generation From Text : Grapher(2022) 논문 요약정리 - 202127288 장석규
(Igor Melnyk, Pierre Dognin, Payel Das)


## [Abstract]

- end-to-end multi-stage Knowledge Graph(KG) generation system from textual inputs

**Two stages** 

- The graph nodes are generated using pretrained language model

- The relationships are generated using available entity information

**Challenges**

- non-unique graph representation

- complex node and edge structure

## [Properties]

- Use sota language models pretrained on large textual corpora (node generation is the key to algorithm performance)
  - fine tuning the pretrained Transformer based language model

- partitioning of graph construction process into two steps ensure efficiency(generate each node and edge only once : Unique model)
  
<img width="650" alt="image" src="https://github.com/wkdtjrrb/knowledge_graph/assets/103736979/982f1e93-e925-4b70-a6be-44a4754edeb1">


- end to end trainable(avoid the need of any external NLP pipelines)

## [Method]

<img width="650" alt="image" src="https://github.com/wkdtjrrb/knowledge_graph/assets/103736979/19f9dec8-cee9-42f5-babf-79eff4679abc">

**1. Node Generation: Text Nodes**
- objective: generate a set of unique nodes
  
- Use pre-trained encoder-decoder language model(PLM, sequence-to-sequence)

- this module additionally supplies node features for the downstream task of edge generation

- greedy decode the generated string and utilize the seperation tokens 
<NODE_SEP> to delineate the node boundaries and mean-pool the hidden states of the decoder's last layer
  - <NODE_SEP> : 노드의 경계를 구분하는 토큰

- fix upfront the number of generated nodes and fill the missing ones with a special <NO_NODE> token

- This approach ignores that the graph nodes are permutation invariant
permutation invariant(한정된 집합의 요소(여기에서는 노드)를 임의의 순서로 배열해도 결과가 동일한 특성을 가짐)

- node generation using traditional S2S model

<img width="650" alt="image" src="https://github.com/wkdtjrrb/knowledge_graph/assets/103736979/56657806-076f-4693-98e1-eddf5c6dbd9d">

**2. Node Generation : Query Nodes**
   
2.1 LEARNALBE NODE QUERIES

- the decoder receives as input a set of learnable node queries, represented as an embedding matrix

- disable causal masking to let Transformer attend to all the queries simultaneously

2.2 PERMUTATION MATRIX

- avoid the system to memorize the particular target node order and enable permutation-invariance

- the logits and features
  
<img width="421" alt="image" src="https://github.com/wkdtjrrb/knowledge_graph/assets/103736979/459ce105-eadb-45ad-81ee-25b4ad1385e4">

s = 1,...,S / P : permutation matrix obtained using bipartite matching algorithm(이분 매칭 알고리즘) between the target and the greedy- decoded nodes

- node generation using learned query vectors
  
<img width="650" alt="image" src="https://github.com/wkdtjrrb/knowledge_graph/assets/103736979/e0c685fc-f15e-4101-948f-ab87309f4c3c">

**3. Edge Generation**

- use the generated set of node features in this module for the edge generation
  - given a pair of node features, a prediction head decides the existence of an edge
  
- Option 1: use a head similar to the one in the **Node Generation : Query Nodes** to generate edges as as a sequence of tokens.
  - generate unseen edges during trainning, at the rsik of not matching the target edge token sequence exactly

<img width="380" alt="image" src="https://github.com/wkdtjrrb/knowledge_graph/assets/103736979/41aa1532-7e19-4264-b035-ba5f2098b35f">

  
- Option 2: use a classification head to predict the edges
  - if the set of possible relationships is fixed and known, this option is efficient and accurate

<img width="383" alt="image" src="https://github.com/wkdtjrrb/knowledge_graph/assets/103736979/b6ae5e04-c2c7-49d8-8ebd-76c3657b1cff">


**4. Imbalance Edge Distribution**

- small savings : ignoring self-edges, igonring edges with the node <NO_NODE>

- <NO_EDGE> : token that is given when there is no edge between two nodes

- the number of acutal edges is small and <NO_EDGE> is large
  - the generation and classification task is imbalnaced towards the <NO_EDGE> token/class
  - to solve this problem, we can use modification of cross-entropy loss and change in the training pardigm
 
4.1 FOCAL LOSS

- replace the CE loss with F loss
  - F loss : down-weight the CE loss for  ell classified sample and increase the CE loss for         mis-classified ones(클래스 불균형에 대처)
 
    <img width="317" alt="image" 
      src="https://github.com/wkdtjrrb/knowledge_graph/assets/103736979/c09389b2-19b9-4ed6-92d3-70818362703d">

4.2 SPARSE LOSS

- sparsifying the adjacency matrix to remove most of the <NO_EDGE> edges, re-balancing the classes artificially

## [LIMITATION]
1. the computational complexity of edge generation

2. Since nodes are generated using trasnformer-based models, the complexity of the attention mechanism leads to the limit on the size of the input text the system can handle

3. Only applied to ENGLISH domain



