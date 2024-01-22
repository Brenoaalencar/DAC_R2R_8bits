# DAC_R2R_8bits
Circuito DAC R-2R 8 bits e geração de três sinais (onda quadrada, triangular e senoidal)

No projeto é utilizado um DAC - Digital-to-Analog Converter - de 8 bits e topologia R-2R com resistores de 10 kohms de
tolerância 1% para gerar os sinais necessários. Para a geração de sinais será usada uma placa de desenvolvimento Nucleo Stm32 F103RB e o ambiente de programação utilizado foi o Keil Studio.

## Esquemáticos
Com o objetivo de facilitar a montagem física do circuito e auxiliar na documentação
do projeto, foi elaborado um esquemático simplificado do DAC projetado. Note que as
entradas digitais D3 a D10 fazem referência às portas usadas na placa Núcleo para produzir
os sinais desejados. Além disso, a quantidade de entradas deve ser a mesma que o número de
bits, portanto, para a elaboração de um DAC com 8 bits foram necessárias 8 entradas digitais.

<div align="center">
<img src ="https://github.com/Brenoaalencar/DAC_R2R_8bits/assets/72100554/30cf34fa-27c3-4e28-bf4b-c592e563564c" width="600px"/>
</div>

## Fluxograma
Uma importante etapa que precede a elaboração do código de geração das ondas
desejadas consiste na realização de um fluxograma que represente e facilite a organização da
estratégia de programação usada, representada na imagem abaixo.
Analisando o diagrama de blocos em questão percebe-se que o evento que leva à
alternância entre as formas de onda ocorre quando o botão de usuário é pressionado.

<div align="center">
<img src ="https://github.com/Brenoaalencar/DAC_R2R_8bits/assets/72100554/0681fb45-45f7-4c2a-8058-a5e97faaccab" width="400px"/>
</div>

## Software
Com o fluxograma criado, desenvolve-se o código em C++ no ambiente Keil Studio.

```C++
#include "mbed.h"
#define tempo 0.001    // define o periodo da onda triangular
#define tempoq 0.26    // define o periodo da onda quadrada
#define temposen 0.002 // define o periodo da onda senoidal
BusOut saida(D3, D4, D5, D6, D7, D8, D9,
             D10); // Recurso BusOut nos pinos ligados ao DAC R-2R 8-bit
InterruptIn botao(USER_BUTTON); // define que o botao utilizado para mudar o
                                // sinal sera o USER_BUTTON

int g = 0; // define a variavel g que sera utilizada para escolher o sinal a ser
           // enviado ao BusOut

// Lookup table

uint8_t seno[] = {0x80,0x83,0x86,0x89,0x8c,0x8f,0x92,0x95, //lista de valores hexadecimais que ao serem enviados ao BusOut definem
0x98,0x9c, 0x9f, 0xa2,0xa5,0xa8,0xab,0xae,// uma ondasenoidal
0xb0, 0xb3, 0xb6, 0xb9, 0xbc, 0xbf, 0xc1,0xc4,
0xc7,0xc9, 0xcc, 0xce, 0xd1,0xd3,0xd5,0xd8,
0xda, 0xdc, 0xde, 0xe0,0xe2,0xe4,0xe6,0xe8,
0xea, 0xeb, 0xed, 0xef, 0xf0,0xf2,0xf3,0xf4,
0xf6,0xf7, 0xf8,0xf9,0xfa, 0xfb, 0xfb,0xfc,
0xfd, 0xfd, 0xfe, 0xfe, 0xfe, 0xff, 0xff,0xff,
0xff,0xff,0xff,0xff,0xfe,0xfe, 0xfd, 0xfd,
0xfc, 0xfc, 0xfb, 0xfa, 0xf9, 0xf8,0xf7,0xf6,
0xf5,0xf4,0xf2,0xf1,0xef,0xee, 0xec, 0xeb,
0xe9,0xe7, 0xe5, 0xe3,0xe1,0xdf, 0xdd, 0xdb,
0xd9, 0xd7, 0xd4, 0xd2,0xcf,0xcd,0xca, 0xc8,
0xc5,0xc3, 0xc0, 0xbd, 0xba, 0xb8, 0xb5,0xb2,
0xaf,0xac,0xa8,0xa6,0xa3,0xa1,0x9d,0x9a,
0x97,0x94,0x91,0x8e, 0x8a, 0x87,0x84,0x81,
0x7e, 0x7b, 0x78, 0x75,0x71,0x6e, 0x6b, 0x68,
0x65,0x62,0x5f, 0x5c, 0x59,0x56,0x53,0x50,
0x4d,0x4a, 0x47,0x45,0x42,0x3f,0x30,0x3a,
0x37,0x35,0x32,0x30,0x2d, 0x2b, 0x28,0x26,
0x24,0x22,0x20,0x1e,0x1c,0x1a,0x18,0x16,
0x14,0x13,0x11,0x10,0xe,0xd, 0xb, 0xa,
0x9,0x8,0x7,0x6, 0x5, 0x4, 0x3, 0x3,
0x2,0x2,0x1,0x1,0x0,0x0,0x0, 0x0,
0x0,0x0,0x0,0x1,0x1,0x1,0x2,0x2,
0x3,0x4,0x4,0x5,0x6, 0x7, 0x8, 0x9,
0xb, 0xc, 0xd, 0xf, 0x10,0x12,0x14,0x15,
0x17,0x19,0x1b,0x1d, 0x1f,0x21,0x23,0x25,
0x27,0x2a, 0x2c, 0x2e,0x31,0x33,0x36,0x38,
0x3b, 0x3e, 0x40,0x43,0x46,0x49,0x4c, 0x4f,
0x51,0x54,0x57, 0x5a, 0x5d, 0x60, 0x63, 0x67,
0x6a, 0x6d, 0x70,0x73,0x76,0x79, 0x7c, 0x80
};

void incrementa(void);
int main() {
  botao.rise(&incrementa); // quando o botão for
  while (1) {
    switch (g) { // verifica qual o valor de g
    case 0:      // se g=0, o sinal retornado será constante e com valor nulo.
      saida = 0;
      break;
    case 1: // se g=1, o sinal retornado será uma onda triangular
      for (int i = 0; i <= 255;
           i++) { // plota a parte "crescente" da onda triangular
        saida = i;
        wait(tempo);
      }
      for (int i = 255; i > 0;
           i--) { // plota a parte "decrescente" da onda triangular
        saida = i;
        wait(tempo);
      }
      break;
    case 2:         // se g=2, o sinal retornado será uma onda quadrada
      saida = 0xff; // valor pico da onda quadrada
      wait(tempoq);
      saida = 0;
      wait(tempoq); // valor vale da onda quadrada
      break;
    case 3: // se g=3, o sinal retornado será uma onda senoidal
      for (int u = 0; u <= 255; u++) {
        saida = seno[u];
        wait(temposen);
      }
      break;
    }
  }
}

void incrementa(void) // funcao define que toda vez que ela for chamada o g
                      // aumentará em um, e caso g for maior que 3,
// ele voltara a valer 1
{
  if (g > 3) { // se g for maior que 1, faz ele voltar a valer 1
    g = 1;
    saida = 0; // faz com que o código retorna o sinal equivalente a g=1, que é
               // um valor constante e nulo
  } else {
    g++; // aumenta g em 1
  }
}

```
## Montagem e Resultado Final
Adotando como ponto de partida o esquemático realizado no Proteus, foi possível
realizar a montagem do DAC de 8 bits por meio de um circuito físico, representado na
imagem a seguir.

<div align="center">
<img src ="https://github.com/Brenoaalencar/DAC_R2R_8bits/assets/72100554/6514405a-d2a6-43ee-ae69-8e53451a8eb5" width="400px"/>
</div>

A seguir, ondas quadrada triangular e senoidal geradas pelo DAC.

<div align="center">
<img src ="https://github.com/Brenoaalencar/DAC_R2R_8bits/assets/72100554/7cd98e7b-8d4e-4fde-98c2-61e6869da2c7" width="400px"/>
</div>

Onda quadrada.

<div align="center">
<img src ="https://github.com/Brenoaalencar/DAC_R2R_8bits/assets/72100554/5ffb2198-e6dd-4252-8fbf-c9c7ce7239c8" width="400px"/>
</div>

Onda triangular.

<div align="center">
<img src ="https://github.com/Brenoaalencar/DAC_R2R_8bits/assets/72100554/a51fd874-fa69-4f8e-9435-bbaac8d8cf3f" width="400px"/>
</div>

Onda senoidal gerada com uma lookup table.
