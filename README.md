# YOLOHardwareCalculatorSite

# Planejamento de Infraestrutura e Análise Técnica para Sistemas de Vigilância Inteligente

## Uma Avaliação entre YOLOv8, YOLO11 e YOLO26

A implementação de sistemas de vigilância baseados em visão computacional evoluiu de simples monitores de movimento para complexas infraestruturas de análise comportamental e identificação de padrões em tempo real.

O núcleo dessa transformação reside na família de modelos YOLO (*You Only Look Once*), que revolucionou a detecção de objetos ao tratar o problema como uma única tarefa de regressão, permitindo velocidades de processamento sem precedentes.

No cenário atual, a escolha entre as versões **YOLOv8**, **YOLO11** e a recente **YOLO26** não é apenas uma questão de preferência algorítmica, mas uma decisão estratégica de engenharia que impacta diretamente os custos de hardware, a latência de resposta e a capacidade de expansão do sistema em ambientes de produção de alta densidade.

---

# Evolução das Arquiteturas e Seleção de Modelos para Vigilância Paralela

A trajetória das arquiteturas Ultralytics demonstra um foco crescente na eficiência de parâmetros e na facilidade de implantação.

O **YOLOv8**, lançado no início de 2023, estabeleceu um novo padrão da indústria ao introduzir uma estrutura sem âncoras (*anchor-free*) e um *head* de detcção desacoplado, o que permitiu uma melhor generalização em diversas proporções de imagem.

Esta versão consolidou-se como o cavalo de batalha da vigilância devido ao seu ecossistema maduro e suporte extensivo da comunidade.

Entretanto, o lançamento do **YOLO11** em setembro de 2024 trouxe refinamentos significativos na extração de características.

A substituição dos blocos C2f pelos módulos C3k2 e a introdução da atenção espacial C2PSA permitiram que o modelo capturasse detalhes mais finos com uma contagem de parâmetros reduzida.

A análise técnica indica que o **YOLO11m**, por exemplo, utiliza **22% menos parâmetros** do que seu antecessor **YOLOv8m**, enquanto entrega uma precisão média superior (*mAP*) no dataset COCO.

Para sistemas de vigilância que exigem a detcção de objetos pequenos ou parcialmente ocluídos, como pedestres distantes em ambientes urbanos, o YOLO11 apresenta uma vantagem qualitativa evidente.

Em janeiro de 2026, a Ultralytics apresentou o **YOLO26**, uma arquitetura especificamente projetada para resolver os gargalos de implantação em dispositivos de borda (*edge*) e processadores de baixo consumo.

A inovação disruptiva do YOLO26 reside na eliminação do processamento de **Supressão Não Máxima (NMS)** através de um design nativo de ponta a ponta (*end-to-end*).

Em fluxos de vídeo simultâneos, o NMS tradicional pode consumir recursos consideráveis da CPU após a inferência da GPU, criando uma variabilidade na latência que prejudica a previsibilidade do sistema.

Ao produzir predições finais diretamente, o YOLO26 simplifica drasticamente o pipeline de produção e garante um desempenho mais estável em sistemas com dezenas de câmeras.

---

# Comparação de Desempenho e Métricas de Eficiência

Para determinar o hardware necessário, é fundamental analisar as métricas de latência e consumo de recursos.

A tabela abaixo resume o desempenho das variantes mais comuns em ambientes de produção.

## Tabela Comparativa de Modelos

| Modelo | Resolução (imgsz) | mAP 50-95 | Parâmetros (M) | FLOPs (B) | Velocidade CPU ONNX (ms) | Velocidade T4 TensorRT (ms) |
|---|---:|---:|---:|---:|---:|---:|
| YOLOv8n | 640 | 37.3 | 3.2 | 8.7 | 80.4 | 0.99 |
| YOLO11n | 640 | 39.5 | 2.6 | 6.5 | 56.1 | 1.50 |
| YOLO26n | 640 | 40.9 | 2.4 | 5.4 | 38.9 | 1.70 |
| YOLOv8s | 640 | 44.9 | 11.2 | 28.6 | 128.4 | 1.89 |
| YOLO11s | 640 | 47.0 | 9.4 | 21.5 | 90.0 | 2.50 |
| YOLO26s | 640 | 48.6 | 9.5 | 20.7 | 87.2 | 2.50 |
| YOLOv8m | 640 | 50.2 | 25.9 | 78.9 | 197.5 | 3.53 |
| YOLO11m | 640 | 51.5 | 20.1 | 68.0 | 183.2 | 4.70 |
| YOLO26m | 640 | 53.1 | 20.4 | 68.2 | 220.0 | 4.70 |

A análise desses dados sugere que, embora o YOLOv8 mantenha velocidades de inferência competitivas em GPUs de alto desempenho como a NVIDIA T4, o YOLO11 e o YOLO26 oferecem uma relação **mAP por parâmetro significativamente melhor**.

O YOLO26, em particular, demonstra uma aceleração de até **43% em inferências baseadas apenas em CPU**, tornando-o a escolha recomendada para implantações que utilizam servidores sem GPUs dedicadas ou dispositivos como Raspberry Pi 5 e NVIDIA Jetson Orin Nano.

---

# Relação entre Quantidade de Câmeras e Recursos Computacionais

A escalabilidade de um sistema de vigilância por vídeo não é estritamente linear devido aos custos fixos de gerenciamento de *threads*, decodificação de fluxos RTSP e transferências de memória entre host e dispositivo.

Cada câmera adicional introduz três cargas principais ao sistema:

1. O *bitstream* de vídeo que precisa ser decodificado  
2. O pré-processamento da imagem (*redimensionamento e normalização*)  
3. A inferência da rede neural

---

# Gerenciamento de Memória de Vídeo (VRAM)

A VRAM é frequentemente o recurso mais escasso em servidores de produção.

O consumo de memória é composto por:

- Peso do modelo (fixo)
- Buffers de ativação (escalam com *batch size* e resolução)

Para maximizar a eficiência, os sistemas de produção utilizam o processamento em lote (*batch inference*), agrupando frames de múltiplas câmeras em uma única operação de inferência na GPU.

A estimativa de VRAM necessária para um sistema multi-câmera pode ser modelada pela soma da memória base do modelo e o custo incremental por *stream*.

Modelos maiores, como variantes **Large** e **Extra-Large**, exigem fatias substancialmente maiores de VRAM para manter buffers de ativação estáveis.

## Hardware Recomendado por Escala

| Hardware Sugerido | VRAM Total | Capacidade de Câmeras (1080p @ 15–20 FPS) | Configuração Recomendada |
|---|---:|---:|---|
| NVIDIA Jetson Orin Nano | 8 GB | 2 – 4 | YOLO11n/26n + TensorRT FP16 |
| NVIDIA RTX 3060 / 4060 | 12 GB | 8 – 12 | YOLO11s + NVDEC |
| NVIDIA RTX 3090 / 4090 | 24 GB | 20 – 30 | YOLO11m + batch processing |
| NVIDIA RTX A6000 | 48 GB | 40 – 60 | YOLO11l/26l |
| NVIDIA A100 80GB | 80 GB | 80 – 100+ | DeepStream SDK em escala empresarial |

A evidência empírica indica que a **decodificação de vídeo costuma ser o primeiro gargalo**.

Uma GPU NVIDIA RTX A6000, por exemplo, suporta tecnicamente a decodificação de até **37 fluxos 1080p a 30 FPS** utilizando H.264, independentemente da carga de inferência.

Se a taxa for reduzida para 15 FPS, essa capacidade pode praticamente dobrar.

---

# Gargalos de CPU e Decodificação de Hardware

Embora a inferência ocorra na GPU, a CPU desempenha papel crítico em:

- Recebimento de pacotes RTSP
- *Demuxing* dos streams
- Agendamento das tarefas de pré-processamento

Se o pipeline não for otimizado (por exemplo, OpenCV simples em vez de NVDEC ou QuickSync), a CPU pode atingir **100% de uso com apenas 4 ou 5 câmeras**.

Para sistemas com **30+ câmeras simultâneas**, a arquitetura deve priorizar:

- NVIDIA DeepStream
- Triton Inference Server

O **DeepStream**, em particular, mantém os quadros na memória da GPU do início ao fim, eliminando transferências dispendiosas via PCIe.

---

# Complexidade Computacional Adicional do YOLO Pose

A integração da estimativa de pose humana (*Pose Estimation*) representa uma mudança de paradigma.

Em vez de apenas detectar caixas delimitadoras, o sistema passa a compreender a estrutura corporal humana.

O modelo detecta simultaneamente:

- Pessoa
- 17 pontos-chave (*keypoints*) do COCO

Exemplo:

- Ombros
- Cotovelos
- Pulsos
- Quadris
- Joelhos
- Tornozelos

---

# O Custo da Estimativa de Pose

Ao contrário do que se imagina, o YOLO Pose não executa dois modelos distintos.

Ele utiliza uma única passagem direta (*single forward pass*) com um *head* especializado para regredir:

- Bounding boxes
- Coordenadas dos keypoints

Ainda assim, existe um custo computacional claro.

## Impactos principais

### 1. Aumento de parâmetros e FLOPs

Há um aumento de aproximadamente **15% a 20% nos GFLOPs**.

### 2. Latência de inferência

Exemplo:

- YOLO11n Detect → ~1.5 ms
- YOLO11n-Pose → ~1.8–2.0 ms

Quando multiplicado por dezenas de câmeras, esse custo se torna relevante.

### 3. Pós-processamento e memória

Cada pessoa detectada exige:

- 51 valores adicionais  
(17 pontos × 3 valores)

Isso aumenta:

- largura de banda de memória
- overhead da CPU
- custo de renderização do esqueleto

---

# Comparativo Detect vs Pose

| Variante | Tarefa | Parâmetros (M) | FLOPs (B) | Latência TensorRT (ms) |
|---|---|---:|---:|---:|
| YOLO11n | Detect | 2.6 | 6.5 | 1.5 |
| YOLO11n-Pose | Pose | 2.9 | 7.5 | 1.8 |
| YOLO11s | Detect | 9.4 | 21.5 | 2.5 |
| YOLO11s-Pose | Pose | 10.4 | 23.9 | 2.7 |
| YOLO26s | Detect | 9.5 | 20.7 | 2.5 |
| YOLO26s-Pose | Pose | 10.4 | 23.9 | 2.7 |

A transição para Pose Estimation também afeta a escolha da variante.

Modelos como **YOLO11m** ou **YOLO11l** são frequentemente necessários para garantir precisão em cenários complexos.

---

# Inovações do YOLO26 no Pose Estimation

O YOLO26 introduziu melhorias específicas como o uso de:

## Residual Log-Likelihood Estimation (RLE)

O RLE permite uma localização de keypoints com maior precisão ao modelar a incerteza da predição.

Isso é especialmente útil em:

- análise ergonômica
- detecção de quedas
- ambientes hospitalares

Além disso, sua arquitetura *end-to-end* reduz parte do atraso no pós-processamento.

---

# Servidores vs Dispositivos de Borda

## Servidores GPU Centralizados

Recomendados para:

- instalações acima de 20 câmeras

Exigem:

- CPUs com muitos núcleos (Xeon / EPYC)
- 128 GB RAM DDR5
- Rede 10 Gbps

Ideal para:

- RTX A6000
- RTX 6000 Blackwell

---

## Edge AI

Recomendado para:

- menos de 8 câmeras

Exemplo:

- Jetson Orin Nano
- Jetson Orin NX

Vantagens:

- menor latência
- menor consumo energético
- privacidade local

Limitações:

- memória compartilhada limitada
- necessidade de reduzir resolução
- frame skipping agressivo

---

# Estratégias de Otimização em Produção

## TensorRT (FP16 / INT8)

Pode acelerar a inferência em até **5x**.

### FP16

- quase sem perda de precisão

### INT8

- throughput ainda maior
- exige calibração cuidadosa

---

## ONNX + OpenVINO

Ideal para:

- CPUs Intel

Pode triplicar a velocidade de inferência.

---

## Redução de FPS

Processar:

- 5 FPS
- 10 FPS

em vez de:

- 30 FPS

pode permitir suportar **3x a 6x mais câmeras**.

---

## Região de Interesse (ROI)

Processar apenas:

- portões
- corredores
- áreas críticas

reduz drasticamente os FLOPs.

Também ajuda:

- usar 320x320
- usar 416x416

em vez de 1080p completo.

---

# Conclusões e Recomendações Estratégicas

A análise comparativa entre **YOLOv8**, **YOLO11** e **YOLO26** mostra que o cenário de visão computacional em 2026 favorece modelos que equilibram:

- eficiência de parâmetros
- facilidade de implantação
- baixa latência
- escalabilidade operacional

## Recomendação principal

### YOLO11

Padrão ouro para sistemas com aceleração por GPU.

### YOLO26

Escolha tecnicamente superior para:

- novas implementações
- edge devices
- pipelines sem NMS
- latência determinística

## Deploy recomendado para produção

### NVIDIA DeepStream + YOLO26-Pose + TensorRT FP16

Essa combinação oferece:

- menor overhead
- menor latência
- maior escalabilidade
- melhor estabilidade para dezenas de câmeras

A transição de sistemas reativos para sistemas proativos tornou-se viável, desde que o dimensionamento respeite os limites físicos da decodificação de vídeo, da CPU e da VRAM.

---
