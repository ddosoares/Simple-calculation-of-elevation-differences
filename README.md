# Simple-calculation-of-elevation-differences
The objective of this project is the development of a Python tool capable of processing a sampled altimetric profile (topographic benchmarks) and estimating the elevations of new intermediate points arbitrarily arranged along the terrain.

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

def carregar_dados_iniciais():
    """Retorna os dados oficiais do PDF."""
    dados = {
        'ID Marco': ['01', '02', '03', '04', '05', '06', '07', '08', '09', '10', '11'],
        'Cota': [382, 766, 796, 187, 490, 446, 647, 710, 755, 277, 480]
    }
    return pd.DataFrame(dados)

def calcular_posicoes_marcos(df, espacamento_medido, modo):
    """
    Calcula a posição acumulada no eixo X para cada marco medido.
    Modo 1: Espaçamento acompanha a topografia (distância no terreno).
    Modo 2: Espaçamento é a projeção em um plano horizontal.
    """
    posicoes_x = [0.0]
    
    for i in range(1, len(df)):
        cota_anterior = df.loc[i-1, 'Cota']
        cota_atual = df.loc[i, 'Cota']
        diff_cota = cota_atual - cota_anterior
        
        if modo == 1:
            # Acompanha a topografia: o espaçamento fixo dado é a hipotenusa.
            # Deduzimos o Delta X (projeção horizontal) para o gráfico.
            if abs(diff_cota) > espacamento_medido:
                # Tratamento caso a diferença de cota exceda o espaçamento físico
                delta_x = 0.0 
            else:
                # Formula: raiz(espaçamento_terreno^2 - diff_cotas^2)
                delta_x = np.sqrt(espacamento_medido**2 - diff_cota**2)
        else:
            # Projeção horizontal: o próprio espaçamento já é o Delta X direto.
            delta_x = espacamento_medido
            
        posicoes_x.append(posicoes_x[-1] + delta_x)
        
    df['Posicao_X'] = posicoes_x
    return df

def interpolar_pontos_intermediarios(df_marcos, espacamento_calculo):
    """
    Calcula as cotas e posições dos pontos intermediários arbitrariamente
    espaçados ao longo do eixo X, garantindo a continuidade (sem saltos abruptos).
    """
    x_marcos = df_marcos['Posicao_X'].values
    y_cotas = df_marcos['Cota'].values
    
    x_max = x_marcos[-1]
    
    # Gerando os novos pontos no eixo X com base no espaçamento arbitrário escolhido
    x_calculados = []
    atual = 0.0
    while atual <= x_max:
        x_calculados.append(atual)
        atual += espacamento_calculo
        
    # Interpolação linear para evitar funções descontínuas visualmente
    y_calculados = np.interp(x_calculados, x_marcos, y_cotas)
    
    df_resultado = pd.DataFrame({
        'Posicao_X': x_calculados,
        'Cota_Calculada': y_calculados
    })
    return df_resultado

def plotar_grafico(df_marcos, df_interp):
    """Gera e exibe o gráfico comparativo."""
    plt.figure(figsize=(10, 6))
    
    # Linha contínua + pontos dos marcos originais
    plt.plot(df_marcos['Posicao_X'], df_marcos['Cota'], 'ro-', label='Marcos Medidos', markersize=6, linewidth=2)
    # Pontos intermediários calculados
    plt. Burma = plt.scatter(df_interp['Posicao_X'], df_interp['Cota_Calculada'], color='blue', marker='x', label='Pontos Intermediários', s=30)
    
    plt.title('Comparativo de Topografia: Medido vs. Calculado')
    plt.xlabel('Variação no Eixo X (Metros)')
    plt.ylabel('Cota / Elevação (Metros)')
    plt.grid(True, linestyle='--', alpha=0.6)
    plt.legend()
    plt.savefig('comparativo_topografia.png', dpi=300)
    plt.show()

def main():
    print("--- PROGRAMA DE CÁLCULO E INTERPOLAÇÃO DE COTAS TOPOGRÁFICAS ---")
    df = carregar_dados_iniciais()
    
    # 1. Entrada de dados do usuário via console (com validação robusta tipo do-while simulado)
    while True:
        try:
            esp_medido = float(input("Digite o espaçamento fixo entre os marcos medidos (em metros): "))
            if esp_medido <= 0:
                print("O espaçamento deve ser maior que zero.")
                continue
            break
        except ValueError:
            print("Entrada inválida. Digite um número.")
            
    while True:
        try:
            esp_calculo = float(input("Digite o espaçamento desejado para os novos pontos calculados (em metros): "))
            if esp_calculo <= 0:
                print("O espaçamento deve ser maior que zero.")
                continue
            break
        except ValueError:
            print("Entrada inválida. Digite um número.")

    while True:
        print("\nEscolha o modelo de projeção do espaçamento dos marcos:")
        print("1 - Acompanha a topografia (Distância real medida no terreno)")
        print("2 - Projeção em plano horizontal")
        opcao = input("Opção (1 ou 2): ")
        if opcao in ['1', '2']:
            modo = int(opcao)
            break
        print("Opção inválida. Escolha 1 ou 2.")

    # 2. Processamento dos Dados
    df_marcos = calcular_posicoes_marcos(df, esp_medido, modo)
    df_calculado = interpolar_pontos_intermediarios(df_marcos, esp_calculo)
    
    # 3. Exportação dos Resultados
    df_calculado.to_csv('pontos_calculados.csv', index=False, sep=';')
    print("\n[Sucesso] Arquivo 'pontos_calculados.csv' gerado com êxito.")
    
    # 4. Plotagem Gráfica
    print("[Sucesso] Imagem gráfica 'comparativo_topografia.png' salva.")
    plotar_grafico(df_marcos, df_calculado)

if __name__ == "__main__":
    main()
