# YOLOHardwareCalculatorSite

Planejamento de Infraestrutura e Análise Técnica para Sistemas de Vigilância Inteligente: Uma Avaliação entre YOLOv8, YOLO11 e YOLO26
A implementação de sistemas de vigilância baseados em visão computacional evoluiu de simples monitores de movimento para complexas infraestruturas de análise comportamental e identificação de padrões em tempo real. O núcleo dessa transformação reside na família de modelos YOLO (You Only Look Once), que revolucionou a detecção de objetos ao tratar o problema como uma única tarefa de regressão, permitindo velocidades de processamento sem precedentes.1 No cenário atual, a escolha entre as versões YOLOv8, YOLO11 e a recente YOLO26 não é apenas uma questão de preferência algorítmica, mas uma decisão estratégica de engenharia que impacta diretamente os custos de hardware, a latência de resposta e a capacidade de expansão do sistema em ambientes de produção de alta densidade.3
Evolução das Arquiteturas e Seleção de Modelos para Vigilância Paralela
A trajetória das arquiteturas Ultralytics demonstra um foco crescente na eficiência de parâmetros e na facilidade de implantação. O YOLOv8, lançado no início de 2023, estabeleceu um novo padrão da indústria ao introduzir uma estrutura sem âncoras (anchor-free) e um head de detecção desacoplado, o que permitiu uma melhor generalização em diversas proporções de imagem.3 Esta versão consolidou-se como o cavalo de batalha da vigilância devido ao seu ecossistema maduro e suporte extensivo da comunidade.3
Entretanto, o lançamento do YOLO11 em setembro de 2024 trouxe refinamentos significativos na extração de características. A substituição dos blocos C2f pelos módulos C3k2 e a introdução da atenção espacial C2PSA permitiram que o modelo capturasse detalhes mais finos com uma contagem de parâmetros reduzida.3 A análise técnica indica que o YOLO11m, por exemplo, utiliza 22% menos parâmetros do que seu antecessor YOLOv8m, enquanto entrega uma precisão média superior (mAP) no dataset COCO.3 Para sistemas de vigilância que exigem a detecção de objetos pequenos ou parcialmente ocluídos, como pedestres distantes em ambientes urbanos, o YOLO11 apresenta uma vantagem qualitativa evidente.7
Em janeiro de 2026, a Ultralytics apresentou o YOLO26, uma arquitetura especificamente projetada para resolver os gargalos de implantação em dispositivos de borda (edge) e processadores de baixo consumo.5 A inovação disruptiva do YOLO26 reside na eliminação do processamento de Supressão Não Máxima (NMS) através de um design nativo de ponta a ponta (end-to-end).10 Em fluxos de vídeo simultâneos, o NMS tradicional pode consumir recursos consideráveis da CPU após a inferência da GPU, criando uma variabilidade na latência que prejudica a previsibilidade do sistema.11 Ao produzir predições finais diretamente, o YOLO26 simplifica drasticamente o pipeline de produção e garante um desempenho mais estável em sistemas com dezenas de câmeras.11
Comparação de Desempenho e Métricas de Eficiência
Para determinar o hardware necessário, é fundamental analisar as métricas de latência e consumo de recursos. A tabela abaixo resume o desempenho das variantes mais comuns em ambientes de produção.

Modelo
Resolução (imgsz)
mAP 50-95
Parâmetros (M)
FLOPs (B)
Velocidade CPU ONNX (ms)
Velocidade T4 TensorRT (ms)
YOLOv8n
640
37.3
3.2
8.7
80.4
0.99 1
YOLO11n
640
39.5
2.6
6.5
56.1
1.50 3
YOLO26n
640
40.9
2.4
5.4
38.9
1.70 10
YOLOv8s
640
44.9
11.2
28.6
128.4
1.89 1
YOLO11s
640
47.0
9.4
21.5
90.0
2.50 3
YOLO26s
640
48.6
9.5
20.7
87.2
2.50 10
YOLOv8m
640
50.2
25.9
78.9
197.5
3.53 1
YOLO11m
640
51.5
20.1
68.0
183.2
4.70 3
YOLO26m
640
53.1
20.4
68.2
220.0
4.70 10

A análise desses dados sugere que, embora o YOLOv8 mantenha velocidades de inferência competitivas em GPUs de alto desempenho como a NVIDIA T4, o YOLO11 e o YOLO26 oferecem uma relação mAP por parâmetro significativamente melhor.3 O YOLO26, em particular, demonstra uma aceleração de até 43% em inferências baseadas apenas em CPU, tornando-o a escolha recomendada para implantações que utilizam servidores sem GPUs dedicadas ou dispositivos como Raspberry Pi 5 e NVIDIA Jetson Orin Nano.1
Relação entre Quantidade de Câmeras e Recursos Computacionais
A escalabilidade de um sistema de vigilância por vídeo não é estritamente linear devido aos custos fixos de gerenciamento de threads, decodificação de fluxos RTSP e transferências de memória entre host e dispositivo. Cada câmera adicional introduz três cargas principais ao sistema: o bitstream de vídeo que precisa ser decodificado, o pré-processamento da imagem (redimensionamento e normalização) e a inferência da rede neural.14
Gerenciamento de Memória de Vídeo (VRAM)
A VRAM é frequentemente o recurso mais escasso em servidores de produção. O consumo de memória é composto pelo peso do modelo (fixo) e pelos buffers de ativação, que escalam com o tamanho do lote (batch size) e a resolução da imagem.17 Para maximizar a eficiência, os sistemas de produção utilizam o processamento em lote, agrupando frames de múltiplas câmeras em uma única operação de inferência na GPU.19
A estimativa de VRAM necessária para um sistema multi-câmera pode ser modelada pela soma da memória base do modelo e o custo incremental por stream. Modelos maiores, como as variantes "Large" ou "Extra-Large", exigem fatias substancialmente maiores de VRAM para manter buffers de ativação estáveis.17

Hardware Sugerido
VRAM Total
Capacidade de Câmeras (1080p @ 15-20 FPS)
Configuração Recomendada
NVIDIA Jetson Orin Nano
8 GB
2 - 4
YOLO11n/26n, otimização TensorRT FP16.23
NVIDIA RTX 3060 / 4060
12 GB
8 - 12
YOLO11s, decodificação via NVDEC.26
NVIDIA RTX 3090 / 4090
24 GB
20 - 30
YOLO11m, alta taxa de FPS, processamento em lote.22
NVIDIA RTX A6000
48 GB
40 - 60
YOLO11l/26l, ideal para múltiplos modelos simultâneos.14
NVIDIA A100 80GB
80 GB
80 - 100+
Escala empresarial, uso intensivo de DeepStream SDK.17

A evidência empírica indica que a decodificação de vídeo costuma ser o primeiro gargalo. Uma GPU NVIDIA RTX A6000, por exemplo, suporta tecnicamente a decodificação de até 37 fluxos 1080p a 30fps utilizando o codec H.264, independentemente da carga de inferência.14 Se a taxa de quadros for reduzida para 15 FPS, a capacidade de decodificação pode ser dobrada, permitindo que a GPU seja mais bem aproveitada para a execução do modelo YOLO.14
Gargalos de CPU e Decodificação de Hardware
Embora a inferência ocorra na GPU, a CPU desempenha um papel crítico no recebimento dos pacotes RTSP, no demuxing dos streams e no agendamento das tarefas de pré-processamento.15 Se o pipeline não for otimizado (ex: utilizando OpenCV simples em vez de decodificação via hardware como NVDEC ou QuickSync), a CPU pode atingir 100% de uso com apenas 4 ou 5 câmeras, impossibilitando a escalabilidade.14
Para sistemas que visam operar com 30 ou mais câmeras simultâneas, a arquitetura deve priorizar frameworks de baixa latência como o NVIDIA DeepStream ou o Triton Inference Server.27 O DeepStream, em particular, permite que os quadros permaneçam na memória da GPU desde a decodificação até a saída final, eliminando as transferências dispendiosas entre o barramento PCIe.14
Complexidade Computacional e Adicional do YOLO Pose
A integração da estimativa de pose humana (Pose Estimation) em um sistema de vigilância representa uma mudança de paradigma, indo além da simples detecção de caixas delimitadoras para a compreensão da estrutura corporal.31 O YOLO Pose detecta simultaneamente a pessoa e seus pontos-chave (keypoints), normalmente os 17 pontos definidos pelo dataset COCO, incluindo articulações como ombros, cotovelos, pulsos, quadris, joelhos e tornozelos.32
O Custo da Estimativa de Pose
Ao contrário do que se possa supor, o YOLO Pose não executa dois modelos distintos; ele utiliza uma única passagem direta (single forward pass) com um head de tarefa específico que regride as coordenadas dos pontos-chave juntamente com as coordenadas da caixa delimitadora.2 No entanto, existe um "imposto" computacional claro para essa funcionalidade extra.
Aumento de Parâmetros e Operações: A arquitetura do YOLOv11-Pose ou YOLO26-Pose adiciona camadas de convolução no head do modelo para gerenciar as coordenadas  e os scores de visibilidade para cada um dos 17 pontos-chave.32 Isso resulta em um aumento de cerca de 15% a 20% nos GFLOPs (bilhões de operações de ponto flutuante) em comparação com as versões de detecção pura.36
Latência de Inferência: Benchmarks comparativos mostram que a variante Pose é invariavelmente mais lenta. Por exemplo, enquanto o YOLO11n leva aproximadamente 1.5ms para detecção em uma GPU T4, o YOLO11n-Pose pode levar cerca de 1.8ms a 2.0ms.3 Embora a diferença pareça pequena para uma única imagem, ela se acumula significativamente quando multiplicada por 50 câmeras a 30 FPS.
Complexidade de Pós-processamento e Memória: O volume de dados de saída aumenta drasticamente. Para cada pessoa detectada, o modelo deve retornar 51 valores adicionais (17 pontos  3 valores:  e visibilidade). Isso exige mais largura de banda de memória para o pós-processamento e pode aumentar o overhead da CPU se a visualização (desenho do esqueleto nos frames) for necessária para gravação.10

Variante do Modelo
Tarefa
Parâmetros (M)
FLOPs (B)
Latência TensorRT (ms)
YOLO11n
Detect
2.6
6.5
1.5 3
YOLO11n-Pose
Pose
2.9
7.5
1.8 36
YOLO11s
Detect
9.4
21.5
2.5 3
YOLO11s-Pose
Pose
10.4
23.9
2.7 36
YOLO26s
Detect
9.5
20.7
2.5 10
YOLO26s-Pose
Pose
10.4
23.9
2.7 36

A transição para o Pose Estimation também afeta a escolha da variante do modelo. O YOLO11m ou YOLO11l são frequentemente necessários para garantir a precisão dos pontos-chave em cenários complexos ou com pessoas sobrepostas, já que as variantes "Nano" podem ter dificuldade em localizar articulações com precisão em resoluções mais baixas ou condições de iluminação desfavoráveis.38
Inovações do YOLO26 no Pose Estimation
O YOLO26 introduziu melhorias específicas para tarefas de pose, como o uso de Residual Log-Likelihood Estimation (RLE). O RLE permite uma localização de pontos-chave de alta precisão ao modelar a incerteza da predição, o que é especialmente útil em aplicações de análise ergonômica ou detecção de quedas em ambientes hospitalares.10 Além disso, a arquitetura end-to-end do YOLO26 mitigou parte do atraso no pós-processamento, permitindo que a estimativa de pose ocorra com uma latência mais próxima da detecção simples do que nas versões anteriores.10
Análise de Hardware para Produção: Servidores vs. Dispositivos de Borda
A escolha do hardware deve ser guiada pelo volume de câmeras e pela necessidade de centralização. Existem duas abordagens principais para o deploy de sistemas YOLO em 2026: a abordagem de servidor centralizado com múltiplas GPUs e a abordagem distribuída em dispositivos de borda.23
Servidores GPU Centralizados
Para instalações de grande porte (acima de 20 câmeras), um servidor centralizado equipado com GPUs profissionais é a solução mais robusta. O uso de placas como a NVIDIA RTX A6000 ou a mais recente RTX 6000 Blackwell permite consolidar o processamento de dezenas de fluxos em uma única máquina, facilitando a manutenção e a integração com sistemas de armazenamento (NVR).14
Nestes ambientes, a largura de banda do barramento PCIe e o throughput da memória RAM do sistema tornam-se fatores críticos. Recomenda-se o uso de processadores com alto número de cores, como a linha Intel Xeon Silver ou AMD EPYC, para gerenciar as threads de entrada RTSP e as operações de I/O de rede.14 Um sistema típico com 50 câmeras pode exigir 128 GB de RAM DDR5 e uma conexão de rede de 10 Gbps para evitar gargalos de ingestão de dados.14
Dispositivos de Borda (Edge AI)
Para locais remotos ou sistemas com poucas câmeras (menos de 8), os dispositivos de borda oferecem uma eficiência energética incomparável. O NVIDIA Jetson Orin Nano ou Orin NX são projetados para rodar modelos YOLO11 e YOLO26 com otimização TensorRT, alcançando taxas de quadros em tempo real com consumo de energia inferior a 25W.23
A vantagem da borda é a redução da latência de rede e a preservação da privacidade, já que os dados brutos de vídeo não precisam sair do local físico.24 No entanto, a escalabilidade é limitada. Tentar processar 8 fluxos RTSP de 1080p em um Jetson Orin Nano simultaneamente pode levar a um esgotamento da memória compartilhada, forçando o uso de resoluções de entrada menores (ex: 320x320) ou saltos de frames agressivos.25
Estratégias de Otimização e Melhores Práticas em Produção
Para atingir o máximo potencial do hardware, várias técnicas de otimização devem ser aplicadas durante o deploy. O simples ato de carregar um modelo .pt do PyTorch e rodar em um loop é insuficiente para sistemas multi-câmera.
Quantização e Precisão de Modelo
A conversão para formatos de inferência otimizados é obrigatória.
TensorRT (FP16/INT8): Em GPUs NVIDIA, o TensorRT pode acelerar a inferência em até 5 vezes em comparação com o PyTorch nativo.45 O uso de meia precisão (FP16) mantém a precisão quase idêntica ao modelo original, enquanto a quantização para INT8 pode dobrar o throughput em hardware compatível, embora exija uma etapa de calibração cuidadosa para evitar degradação do mAP.44
ONNX e OpenVINO: Para inferências em CPUs Intel, o framework OpenVINO pode triplicar a velocidade de processamento, permitindo que servidores sem GPU lidem com alguns fluxos de vídeo leves utilizando o YOLO26n.45
Redução de FPS e Skip-Frames
Em vigilância, o processamento de cada frame (30 FPS) é raramente necessário. Ao processar apenas 5 ou 10 frames por segundo, é possível suportar 3 a 6 vezes mais câmeras no mesmo hardware. O uso de detectores de movimento simples (como Background Subtraction do OpenCV) para ativar a inferência YOLO apenas quando há movimento na cena é outra estratégia eficaz para economizar recursos computacionais.25
Região de Interesse (ROI)
Em vez de processar a imagem inteira de 1080p, o sistema pode ser configurado para analisar apenas áreas críticas (ex: portões, corredores). O redimensionamento agressivo da entrada para 320x320 ou 416x416 pode reduzir os FLOPs necessários pela metade, permitindo que modelos menores rodem em hardware muito limitado com perda aceitável de precisão para objetos grandes.23
Conclusões e Recomendações Estratégicas
A análise técnica comparativa entre YOLOv8, YOLO11 e YOLO26 revela que o cenário de visão computacional em 2026 favorece modelos que equilibram a eficiência de parâmetros com a facilidade de implantação end-to-end. Para um sistema de vigilância baseado em RTSP com foco em estimativa de pose, as conclusões são fundamentais para o sucesso do projeto.
O YOLO11 é recomendado como o padrão de ouro para sistemas que utilizam aceleração por GPU devido ao seu refinamento de atenção espacial e maturidade no ecossistema Ultralytics.3 No entanto, o YOLO26 surge como a escolha técnica superior para novas implementações, especialmente aquelas voltadas para dispositivos de borda ou servidores que buscam eliminar a latência variável do pós-processamento NMS.5
Em relação à escalabilidade, o aumento de recursos computacionais segue uma progressão linear em VRAM, mas é frequentemente limitado por gargalos de I/O e decodificação de vídeo na CPU e nos motores NVDEC.14 O uso de Pose Estimation introduz um adicional de complexidade de aproximadamente 15% em carga computacional e latência, exigindo um planejamento de hardware ligeiramente superior ao de um sistema de detecção simples.36
Para um deploy de produção resiliente, recomenda-se a adoção do NVIDIA DeepStream SDK integrado com o YOLO26-Pose otimizado em TensorRT FP16.10 Esta combinação oferece o pipeline mais eficiente, minimizando transferências de dados e garantindo que o sistema possa escalar para dezenas de câmeras com uma latência determinística e alta precisão na detecção de pontos-chave humanos. A transição de sistemas reativos para proativos é agora viável, desde que o dimensionamento do hardware respeite os limites físicos da decodificação de vídeo e a gestão inteligente de memória de vídeo.
Referências citadas
Ultralytics YOLO Evolution: An Overview of YOLO26, YOLO11, YOLOv8, and YOLOv5 Object Detectors for Computer Vision and Pattern Recognition - arXiv, acessado em maio 7, 2026, https://arxiv.org/html/2510.09653v3
YOLOv11 Demystified: A Practical Guide to High-Performance Object Detection - arXiv, acessado em maio 7, 2026, https://arxiv.org/html/2604.03349v1
YOLOv8 vs YOLO11: A Comprehensive Technical Comparison of Real-Time Vision Models, acessado em maio 7, 2026, https://docs.ultralytics.com/compare/yolov8-vs-yolo11/
Ultralytics YOLO26 vs YOLO11 vs YOLOv8: Which one should you use?, acessado em maio 7, 2026, https://www.ultralytics.com/blog/ultralytics-yolo26-vs-yolo11-vs-yolov8-which-one-should-you-use
YOLO26 vs YOLOv8: Advancements in Next-Generation Object Detection - Ultralytics YOLO, acessado em maio 7, 2026, https://docs.ultralytics.com/compare/yolo26-vs-yolov8/
YOLO11 vs YOLOv8: A Comprehensive Technical Comparison of Real-Time Vision Models, acessado em maio 7, 2026, https://docs.ultralytics.com/compare/yolo11-vs-yolov8/
YOLO11 vs YOLOv8 vs YOLO26: Full Comparison (2026) - Ultralytics, acessado em maio 7, 2026, https://www.ultralytics.com/blog/comparing-ultralytics-yolo11-vs-previous-yolo-models
YOLO11 vs YOLOv10: A Comprehensive Technical Comparison of Real-Time Object Detectors - Ultralytics YOLO, acessado em maio 7, 2026, https://docs.ultralytics.com/compare/yolo11-vs-yolov10/
YOLOv26 Explained Simply: The Object Detector Built for the Real World - Medium, acessado em maio 7, 2026, https://medium.com/@harikrishnananu2003/yolov26-explained-simply-the-object-detector-built-for-the-real-world-ceb9b3693c57
Ultralytics YOLO26, acessado em maio 7, 2026, https://docs.ultralytics.com/models/yolo26/
Ultralytics YOLO26: The new standard for edge-first Vision AI, acessado em maio 7, 2026, https://www.ultralytics.com/blog/ultralytics-yolo26-the-new-standard-for-edge-first-vision-ai
YOLO11 vs PP-YOLOE+: A Technical Comparison of Real-Time Detectors - Ultralytics YOLO, acessado em maio 7, 2026, https://docs.ultralytics.com/compare/yolo11-vs-pp-yoloe/
YOLO26: YOLO Model for Real-Time Vision AI [2026] - Roboflow Blog, acessado em maio 7, 2026, https://blog.roboflow.com/yolo26/
How many RTSP stream rackserver CPU can take? - NVIDIA Developer Forums, acessado em maio 7, 2026, https://forums.developer.nvidia.com/t/how-many-rtsp-stream-rackserver-cpu-can-take/357779
High CPU usage, but low GPU usage · Issue #19823 - GitHub, acessado em maio 7, 2026, https://github.com/ultralytics/ultralytics/issues/19823
SOTA method for optimizing YOLO inference with multiple RTSP streams? - Reddit, acessado em maio 7, 2026, https://www.reddit.com/r/computervision/comments/1otfoo3/sota_method_for_optimizing_yolo_inference_with/
What is the optimal GPU memory size for YOLO object detection models, and is the RTX A6000 ADA's 48 GB sufficient? - Massed Compute, acessado em maio 7, 2026, https://massedcompute.com/faq-answers/?question=What%20is%20the%20optimal%20GPU%20memory%20size%20for%20YOLO%20object%20detection%20models,%20and%20is%20the%20RTX%20A6000%20ADA%27s%2048%20GB%20sufficient?
How To Calculate GPU VRAM Requirements for an Large-Language Model, acessado em maio 7, 2026, https://apxml.com/posts/how-to-calculate-vram-requirements-for-an-llm
If you have some VRAM to spare, don't sleep on the Batch Size setting. It lets you generate batches of images MUCH faster in some situations. : r/StableDiffusion - Reddit, acessado em maio 7, 2026, https://www.reddit.com/r/StableDiffusion/comments/113v4gk/if_you_have_some_vram_to_spare_dont_sleep_on_the/
How to dynamically estimate maximum number of cameras my GPU can handle for YOLOv8 inference? - Stack Overflow, acessado em maio 7, 2026, https://stackoverflow.com/questions/79852884/how-to-dynamically-estimate-maximum-number-of-cameras-my-gpu-can-handle-for-yolo
Configure YOLOv8 for GPU: Accelerate Object Detection | DigitalOcean, acessado em maio 7, 2026, https://www.digitalocean.com/community/tutorials/yolov8-for-gpu-accelerate-object-detection
GPU for YOLO : r/computervision - Reddit, acessado em maio 7, 2026, https://www.reddit.com/r/computervision/comments/1mgjz47/gpu_for_yolo/
Estimating Computational Power for 75 AI-Powered Cameras - YOLO - Ultralytics, acessado em maio 7, 2026, https://community.ultralytics.com/t/estimating-computational-power-for-75-ai-powered-cameras/647
Build Custom AI Video Surveillance with YOLO, ByteTrack, BoT-SORT & DeepSORT — 2026 Guide | by Fora Soft, acessado em maio 7, 2026, https://forasoft.medium.com/build-custom-ai-video-surveillance-with-yolo-bytetrack-bot-sort-deepsort-2026-guide-fd55503d618d
Running Multiple RTSP Streams (Up to 4) with YOLOv11n on Raspberry Pi 5 · Issue #23508, acessado em maio 7, 2026, https://github.com/ultralytics/ultralytics/issues/23508
System Hardware Requirements for YOLO in 2025 - ProX PC, acessado em maio 7, 2026, https://www.proxpc.com/blogs/system-hardware-requirements-for-yolo-in-2025
Processing multiple rtsp streams for yolo inference : r/computervision - Reddit, acessado em maio 7, 2026, https://www.reddit.com/r/computervision/comments/1p93ds2/processing_multiple_rtsp_streams_for_yolo/
Can an A100 GPU handle 8 parallel YOLOv11-L real-time inference streams at 1080p 30FPS? - NVIDIA Developer Forums, acessado em maio 7, 2026, https://forums.developer.nvidia.com/t/can-an-a100-gpu-handle-8-parallel-yolov11-l-real-time-inference-streams-at-1080p-30fps/351241
DeepStream-Yolo/docs/benchmarks.md at master - GitHub, acessado em maio 7, 2026, https://github.com/marcoslucianops/DeepStream-Yolo/blob/master/docs/benchmarks.md
NVIDIA DeepStream SDK Developer Guide : Performance, acessado em maio 7, 2026, https://docs.nvidia.com/metropolis/deepstream/5.0/dev-guide/DeepStream_Development_Guide/deepstream_performance.html
Comparing Ultralytics YOLO26 to other YOLO Models for Pose Estimation, acessado em maio 7, 2026, https://www.ultralytics.com/blog/ultralytics-yolo26-vs-other-ultralytics-yolo-models-for-pose-estimation
Controlled comparative study of YOLOv8-Pose, YOLOv11-Pose, and Detectron2 for vertebrae detection and keypoint estimation - PMC, acessado em maio 7, 2026, https://pmc.ncbi.nlm.nih.gov/articles/PMC13098965/
What Is Pose Estimation? Keypoint Detection Explained [2026] | Datature Blog, acessado em maio 7, 2026, https://datature.io/blog/what-is-pose-estimation-keypoint-detection-explained-2026
A Monocular Pose Estimation Framework for Automatic Dragon Fruit Harvesting Using Navel and Stem Keypoints - MDPI, acessado em maio 7, 2026, https://www.mdpi.com/2311-7524/12/4/505
Best Pose Estimation Models & How to Deploy Them, acessado em maio 7, 2026, https://blog.roboflow.com/best-pose-estimation-models/
YOLOv8 vs YOLO26: The Evolution of Ultralytics Real-Time Object Detection, acessado em maio 7, 2026, https://docs.ultralytics.com/compare/yolov8-vs-yolo26/
Ultralytics/YOLO26 - Hugging Face, acessado em maio 7, 2026, https://huggingface.co/Ultralytics/YOLO26
Precision Without Complexity: A Comparative Study of YOLO26 Pose Variants for Distal Arm Landmark Detection - MDPI, acessado em maio 7, 2026, https://www.mdpi.com/2076-3417/16/8/3968
Clarification on Backbone and Limitations for Pose Estimation in YOLOv11 (s/m/l) · Issue #21271 - GitHub, acessado em maio 7, 2026, https://github.com/ultralytics/ultralytics/issues/21271
Controlled comparative study of YOLOv8-Pose, YOLOv11-Pose, and Detectron2 for vertebrae detection and keypoint estimation - ResearchGate, acessado em maio 7, 2026, https://www.researchgate.net/publication/404044232_Controlled_comparative_study_of_YOLOv8-Pose_YOLOv11-Pose_and_Detectron2_for_vertebrae_detection_and_keypoint_estimation
New Release: Ultralytics v8.4.0 - Discussion, acessado em maio 7, 2026, https://community.ultralytics.com/t/new-release-ultralytics-v8-4-0/1747
Recommended NVIDIA GPUs for NVIDIA RTX vWS, acessado em maio 7, 2026, https://docs.nvidia.com/vgpu/sizing/virtual-workstation/latest/gpus-vws.html
Optimal PC Build Specs for GPU inferencing with YOLOv8 · ultralytics · Discussion #15376, acessado em maio 7, 2026, https://github.com/orgs/ultralytics/discussions/15376
YOLOv26 Dual GMSL Camera Image Processing System on Jetson | Seeed Studio Wiki, acessado em maio 7, 2026, https://wiki.seeedstudio.com/ai_roboticsyolov26_dual_camera_system/
Model Benchmarking with Ultralytics YOLO, acessado em maio 7, 2026, https://docs.ultralytics.com/modes/benchmark/
YOLO26: The Edge-First Evolution of Real-Time Object Detection | Datature Blog, acessado em maio 7, 2026, https://datature.io/blog/yolo26-the-edge-first-evolution-of-real-time-object-detection
CPU & GPU requirements to stream 120 rtsp camera streams and apply YoloV8 - Reddit, acessado em maio 7, 2026, https://www.reddit.com/r/computervision/comments/14a02jh/cpu_gpu_requirements_to_stream_120_rtsp_camera/
Introduction - RTSP Human Capture - Mintlify, acessado em maio 7, 2026, https://mintlify.com/itsyourap/rtsp-human-capture/introduction
YOLO11 Edge Vision Module: Hardware, Implementation & Performance, acessado em maio 7, 2026, https://www.ovaga.com/blog/ai/yolo11-edge-vision-module-hardware-implementation-performance
YOLOv10 vs YOLO11: A Deep Dive into Real-Time Object Detection Architectures, acessado em maio 7, 2026, https://docs.ultralytics.com/compare/yolov10-vs-yolo11/
