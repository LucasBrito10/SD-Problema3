# SD-Problema3


## 1. Visão Geral do Projeto

Este projeto foi desenvolvido como parte da avaliação da disciplina de Sistemas Digitais (TEC499) do curso de Engenharia de Computação da Universidade Estadual de Feira de Santana (UEFS). O objetivo principal é finalizar o sistema de redimensionamento de imagens, integrando o hardware desenvolvido nas etapas anteriores com uma aplicação de alto nível com interface gráfica interativa.

Nesta etapa (Etapa 3), o sistema é controlado por uma aplicação completa escrita em linguagem C, executada no processador HPS (ARM). Esta aplicação gerencia a leitura de arquivos de imagem, utiliza a API desenvolvida anteriormente para comunicação com a FPGA e introduz o uso do mouse para seleção dinâmica de regiões de interesse (ROI) na tela.

A base do hardware e do driver utilizados nesta etapa provém do desenvolvimento realizado nas etapas 1 e 2 do projeto.

## 2. Definição do Problema

O tema deste problema é a **Interação Hardware-Software e Aplicações em Espaço de Usuário**.

O objetivo é consolidar o módulo embarcado permitindo que o usuário interaja visualmente com o sistema através de periféricos padrão (mouse e teclado) e arquivos reais.

Este repositório consiste na implementação da aplicação final ("Application Layer") que consome a biblioteca (API) criada na etapa anterior. O foco é a orquestração dos recursos de hardware para permitir o carregamento de imagens BITMAP e a manipulação de zoom em janelas flutuantes definidas pelo usuário.

## 3. Requisitos do Projeto

A seguir estão os requisitos funcionais e não funcionais desenvolvidos durante esta etapa do projeto.

### 3.1. Requisitos Funcionais

* **RF01:** O sistema deve carregar imagens a partir de arquivos no formato BITMAP (.bmp) armazenados no sistema de arquivos.
* **RF02:** O usuário deve ser capaz de selecionar o algoritmo de zoom desejado via interface de texto.
* **RF03:** O sistema deve suportar o uso de um mouse USB para interação.
* **RF04:** Deve ser possível selecionar uma região retangular (janela) na tela definindo dois cantos opostos com o mouse.
* **RF05:** A posição $(x, y)$ do cursor do mouse deve ser exibida em tempo real na interface de texto.
* **RF06:** A janela ampliada deve ser desenhada sobre a imagem original (overlay).
* **RF07:** A operação de **Zoom In** na janela selecionada deve ser controlada pela tecla `+`.
* **RF08:** A operação de **Zoom Out** na janela selecionada deve ser controlada pela tecla `-`, limitada à resolução original.

### 3.2. Requisitos Não Funcionais

* **RNF01:** O código da aplicação deve ser escrito estritamente em linguagem C.
* **RNF02:** O driver do processador (desenvolvido na etapa 2) deve ser link-editado ao código da aplicação principal.
* **RNF03:** Um arquivo de cabeçalho (`.h`) deve ser utilizado para definir os protótipos da API.
* **RNF04:** O sistema deve operar sobre o Linux embarcado na placa **DE1-SoC**.
* **RNF05:** A interação deve ocorrer de forma híbrida: comandos de configuração via terminal e manipulação visual via monitor VGA.

## 4. Fundamentação Teórica

Esta seção detalha a teoria por trás da manipulação de imagens e interação implementadas nesta etapa.

### 4.1. Algoritmos de Zoom (Controlados via Software)

A aplicação C é responsável por configurar o coprocessador para executar um dos seguintes algoritmos:

* **Vizinho Mais Próximo (Nearest Neighbor)**
    * **Funcionamento:** O software envia o comando correspondente à FPGA, que seleciona o pixel mais próximo geometricamente. É o método padrão para pré-visualização rápida na janela de zoom.
    * **Resultado:** Preserva bordas duras, ideal para ampliações onde a nitidez dos pixels originais é desejada (pixel art ou análise técnica).

* **Interpolação Bilinear / Média**
    * **Funcionamento:** Caso selecionado pelo usuário, o software instrui a FPGA a calcular a média ponderada dos pixels vizinhos (ou média de blocos para redução).
    * **Resultado:** Produz imagens mais suaves na janela de zoom, reduzindo o efeito de serrilhado (aliasing).

### 4.2. Janelamento (Windowing)

A técnica de janelamento permite processar apenas uma sub-região da imagem.

* **Definição por Bounding Box**
    * **Teoria:** Ao clicar em dois pontos na tela, $P1(x_1, y_1)$ e $P2(x_2, y_2)$, o software calcula o retângulo que envolve esses pontos.
    * **Funcionamento:** A aplicação lê apenas os pixels contidos neste retângulo da memória original e solicita ao hardware que processe e exiba o resultado em uma camada de sobreposição (overlay), criando o efeito de "lupa" sobre a imagem base.

## 5. Ambiente de Desenvolvimento

### 5.1. Software Utilizado

| Software | Versão | Descrição |
| :--- | :--- | :--- |
| Quartus Prime | 23.1.0 | Síntese do hardware (FPGA). |
| GCC (GNU Compiler Collection) | 9.x+ (ARM) | Compilação do código C e link-edição no HPS. |
| GNU/Linux | Kernel 4.x+ | Sistema Operacional da DE1-SoC. |

### 5.2. Hardware Utilizado

| Componente | Especificação |
| :--- | :--- |
| Kit de Desenvolvimento | Terasic DE1-SoC (Cyclone V). |
| Monitor | Monitor com entrada VGA. |
| Mouse | Mouse USB padrão (conectado à porta USB da placa). |
| Computador | Para acesso SSH e transferência de arquivos. |

## 6. Manual do Usuário

Esta seção descreve como instalar, configurar e operar o sistema final.

### 6.1. Instalação e Configuração

1.  **Clonar o Repositório:**
    ```bash
    git clone [LINK_DO_SEU_REPOSITORIO_ETAPA_3]
    cd <NOME_DA_PASTA>
    ```

2.  **Preparação do Hardware (Se necessário):**
    * Caso a FPGA não esteja programada, utilize o Quartus Prime para carregar o arquivo `.sof` conforme instruções das etapas anteriores.

3.  **Conectando e Transferindo Arquivos para o HPS:**
    * Abra um terminal e conecte-se à placa via SSH:
        ```bash
        ssh aluno@172.65.213.<Digitos finais da Placa>
        # Forneça a senha quando solicitada
        ```
    * Transfira os códigos fonte C, o header, o objeto da biblioteca e o Makefile:
        ```bash
        # No seu computador:
        scp main.c gpu_lib.h gpu_driver.o Makefile aluno@172.65.213.<...>:~/projeto_final/
        ```
    * Transfira uma imagem de teste (Bitmap):
        ```bash
        scp imagem_teste.bmp aluno@172.65.213.<...>:~/projeto_final/
        ```

4.  **Compilação do Software:**
    * No terminal SSH da placa, navegue até a pasta e compile:
        ```bash
        cd projeto_final
        make all  # Compila o C e linka com o driver Assembly
        ```

5.  **Execução:**
    * Certifique-se de que o **Mouse USB** está conectado à placa.
    * Execute o programa com permissões de superusuário (necessário para acesso a drivers de memória e input):
        ```bash
        sudo ./app_zoom
        ```

### 6.2. Comandos de Operação

A interação é dividida entre o Terminal (menus) e o Monitor VGA (mouse e atalhos).

**No Terminal:**

| Opção | Ação |
| :--- | :--- |
| `1` | **Carregar Imagem:** Solicita o nome do arquivo BMP (ex: `imagem.bmp`). |
| `2` | **Selecionar Algoritmo:** Alterna entre Vizinho Mais Próximo, Bilinear, etc. |
| `3` | **Sair:** Encerra a execução do programa. |

**No Monitor (Modo Janela):**

| Ação / Tecla | Função |
| :--- | :--- |
| **Clique Esquerdo (1º)** | Define o primeiro canto da janela de zoom. |
| **Clique Esquerdo (2º)** | Define o canto oposto e desenha a janela. |
| **Tecla `+`** | Aplica **Zoom In** na área selecionada. |
| **Tecla `-`** | Aplica **Zoom Out** na área selecionada (retorna ao original). |
| **Movimento do Mouse** | Atualiza as coordenadas X, Y exibidas no terminal. |

**Notas:**
* **Mouse:** O mouse deve ser conectado à porta USB da DE1-SoC **antes** de iniciar o programa para ser detectado corretamente pelo Linux (`/dev/input/event*`).
* **Limites:** O Zoom Out (tecla `-`) não reduzirá a imagem além de sua resolução original na janela selecionada.
