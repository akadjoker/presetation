# Projeto de Carro Autônomo com Jetson Nano
## Coleta de Dados e Treinamento de Modelo

### Visão Geral do Projeto

Este projeto demonstra a implementação de um sistema de carro autônomo usando a plataforma Jetson Nano. O sistema consiste em dois componentes principais:

1. **Sistema de Coleta de Dados** - Uma interface de controle em tempo real para condução manual enquanto coleta dados de treinamento
2. **Pipeline de Treinamento** - Um modelo de aprendizado profundo para prever ângulos de direção a partir de imagens da câmera

### Configuração de Hardware

- **Jetson Nano** - A unidade principal de processamento
- **Câmera** - Conectada ao Jetson Nano para captura de imagem em tempo real
- **Chassi do Carro** - Plataforma mecânica com motores para direção e propulsão
- **Interface JetCar** - Uma interface personalizada para controlar os motores do carro

### Processo de Coleta de Dados

O sistema de coleta de dados (DataCollect.py) oferece:

- **Controle em Tempo Real** - Dirija o carro manualmente usando controles de teclado:
  - W/S - Acelerar para frente/trás
  - A/D - Virar à esquerda/direita
  - C - Centralizar direção
  - Espaço - Parar
  - T - Alternar coleta de dados
  - Enter - Capturar frames manualmente

- **Coleta de Dados Automatizada**
  - Cada frame capturado é salvo com seu ângulo de direção correspondente
  - Dados organizados em pastas baseadas em sessões com marcação de data/hora
  - Arquivos CSV registram a relação entre imagens e ângulos de direção

- **Feedback Visual**
  - Exibição em tempo real com indicadores de direção e velocidade
  - Visualização do status da coleta de dados
  - Contador de FPS para monitoramento de desempenho

### Organização do Dataset

- Dados armazenados em formato estruturado:
  ```
  dataset/
    session_YYYYMMDD_HHMMSS/
      steering_data.csv
      images/
        frame_YYYYMMDD_HHMMSS_ffffff.jpg
        ...
  ```

- CSV de dados de direção contém:
  - Caminhos para os arquivos de imagem
  - Ângulos de direção correspondentes

- Múltiplas sessões podem ser combinadas para dados de treinamento mais diversos

### Pipeline de Treinamento

O sistema de treinamento (Trainning.py) oferece:

1. **Carregamento e Pré-processamento de Dados**
   - Carrega e combina dados de múltiplas sessões
   - Verifica a existência de imagens e cria datasets válidos
   - Aplica pré-processamento de imagem:
     - Corte para focar na estrada (remove céu/capô)
     - Conversão para espaço de cor YUV para melhor extração de características
     - Aplica desfoque Gaussiano para reduzir ruído
     - Redimensiona para formato 200×66 (dimensões do modelo NVIDIA)

2. **Aumento de Dados**
   - Deslocamentos horizontais e verticais aleatórios (pan)
   - Zoom aleatório para simular diferentes distâncias
   - Ajustes de brilho aleatórios para condições de iluminação
   - Inversões horizontais aleatórias com inversão do ângulo de direção

3. **Balanceamento de Dados**
   - Balanceamento baseado em histograma para evitar viés de direção
   - Limita amostras por bin de ângulo de direção a 300
   - Garante que o modelo não se sobreajuste a andar em linha reta

4. **Arquitetura de Rede Neural**
   - Baseada no modelo de condução end-to-end da NVIDIA
   - Camada de normalização de entrada
   - 5 camadas convolucionais com ativação ELU
   - Normalização em lote após cada camada convolucional
   - 3 camadas totalmente conectadas com dropout para regularização
   - Saída final: previsão de um único ângulo de direção

5. **Processo de Treinamento**
   - Divisão treino/validação (80/20)
   - Tamanho de lote de 32
   - Otimizador Adam com taxa de aprendizado adaptativa
   - Parada antecipada para prevenir sobreajuste
   - Checkpoint de modelo para salvar o modelo com melhor desempenho
   - Redução da taxa de aprendizado em platôs

### Análise de Desempenho do Modelo

- Visualizações de treinamento:
  - Curvas de perda para treinamento e validação
  - Acompanhamento de Erro Absoluto Médio (MAE)
  - Acompanhamento de Erro Quadrático Médio da Raiz (RMSE)

- Visualização de imagens de amostra para verificar a qualidade dos dados
- Análise da distribuição de ângulo de direção
- Verificação de balanceamento antes/depois do pré-processamento

### Principais Características Técnicas

1. **Pipeline de Processamento de Imagem Eficiente**
   - Otimizado para as restrições computacionais do Jetson Nano
   - Processamento de imagem acelerado por CUDA via OpenCV

2. **Arquitetura CNN Inspirada na NVIDIA**
   - Design comprovado para condução end-to-end
   - Aprimorado com normalização em lote e dropout

3. **Técnicas Avançadas de Treinamento**
   - Aumento de dados para melhorar a generalização
   - Balanceamento de histograma para prevenir viés
   - Taxas de aprendizado adaptativas e parada antecipada

4. **Sistemas de Feedback em Tempo Real**
   - Indicadores visuais durante a coleta de dados
   - Métricas de desempenho durante o treinamento

### Lições Aprendidas e Melhorias Futuras

1. **Desafios na Coleta de Dados**
   - Garantir cenários de condução diversos
   - Equilibrar estradas retas vs. curvas
   - Manter comportamento de condução consistente

2. **Otimizações de Treinamento**
   - Ajuste adicional de hiperparâmetros
   - Técnicas adicionais de aumento de dados
   - Transfer learning a partir de modelos pré-treinados

3. **Extensões do Sistema**
   - Adicionar previsão de controle de aceleração
   - Implementar detecção de obstáculos
   - Adicionar detecção de faixa como tarefa auxiliar

### Conclusão

Este projeto demonstra um pipeline completo desde a coleta de dados até o treinamento do modelo para direção autônoma em um veículo em pequena escala. A combinação de coleta manual de dados com aprendizado profundo cria um sistema que pode aprender a navegar baseado apenas em entrada visual.

Aproveitando as capacidades do Jetson Nano, criamos uma plataforma acessível para experimentar com tecnologias de condução autônoma que segue princípios similares a veículos autônomos em escala real.
