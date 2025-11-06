import streamlit as st
import pandas as pd

st.title("⚖️ Simulador de Dosimetria da Pena")
st.write("**Calculadora completa da dosimetria penal conforme Art. 68 do CP**")

# Upload do arquivo
uploaded_file = st.file_uploader("Faça upload do arquivo crimes_cp_final_sem_art68.csv", type=["csv"])

@st.cache_data
def processar_dados_crimes(df):
    """Processa os dados dos crimes para o formato necessário"""
    if df.empty:
        return {}
        
    crimes_dict = {}
    
    for idx, row in df.iterrows():
        artigo_base = row['Artigo_Base'] if pd.notna(row['Artigo_Base']) else ''
        artigo_completo = row['Artigo_Completo'] if pd.notna(row['Artigo_Completo']) else artigo_base
        descricao = row['Descricao_Crime'] if pd.notna(row['Descricao_Crime']) else ''
        pena_min_valor = row['Pena_Minima_Valor'] if pd.notna(row['Pena_Minima_Valor']) else 0
        pena_min_unidade = row['Pena_Minima_Unidade'] if pd.notna(row['Pena_Minima_Unidade']) else 'mês'
        pena_max_valor = row['Pena_Maxima_Valor'] if pd.notna(row['Pena_Maxima_Valor']) else 0
        pena_max_unidade = row['Pena_Maxima_Unidade'] if pd.notna(row['Pena_Maxima_Unidade']) else 'mês'
        tipo_penal = row['Tipo_Penal_Estrutural'] if pd.notna(row['Tipo_Penal_Estrutural']) else 'Crime Base (Caput)'
        
        # Converter para anos
        if pena_min_unidade == 'mês':
            pena_min_anos = pena_min_valor / 12
        elif pena_min_unidade == 'dia':
            pena_min_anos = pena_min_valor / 360
        else:
            pena_min_anos = pena_min_valor
            
        if pena_max_unidade == 'mês':
            pena_max_anos = pena_max_valor / 12
        elif pena_max_unidade == 'dia':
            pena_max_anos = pena_max_valor / 360
        else:
            pena_max_anos = pena_max_valor
        
        # Criar chave única para o crime
        if pd.notna(artigo_completo) and pd.notna(descricao):
            chave = f"{artigo_completo} - {descricao[:80]}..."
            crimes_dict[chave] = {
                'artigo': artigo_completo,
                'artigo_base': artigo_base,
                'descricao_completa': descricao,
                'pena_min': pena_min_anos,
                'pena_max': pena_max_anos,
                'tipo_penal': tipo_penal,
                'pena_min_original': pena_min_valor,
                'pena_max_original': pena_max_valor,
                'unidade_original': pena_min_unidade
            }
    
    return crimes_dict

# Carregar dados baseado no upload
df = pd.DataFrame()
crimes_data = {}

if uploaded_file is not None:
    try:
        # Tenta diferentes codificações
        codificacoes = ['utf-8', 'latin-1', 'iso-8859-1', 'cp1252', 'utf-8-sig']
        
        for encoding in codificacoes:
            try:
                uploaded_file.seek(0)  # Reset file pointer
                df = pd.read_csv(uploaded_file, encoding=encoding)
                st.success(f"✅ Dados carregados com sucesso! (Codificação: {encoding})")
                crimes_data = processar_dados_crimes(df)
                break
            except (UnicodeDecodeError, pd.errors.EmptyDataError):
                continue
        else:
            # Se nenhuma codificação funcionou, tenta com engine python
            try:
                uploaded_file.seek(0)
                df = pd.read_csv(uploaded_file, encoding='latin-1', engine='python')
                st.success("✅ Dados carregados com engine python")
                crimes_data = processar_dados_crimes(df)
            except Exception as e:
                st.error(f"❌ Erro ao carregar arquivo: {e}")
    except Exception as e:
        st.error(f"❌ Erro inesperado: {e}")
else:
    st.info("📁 Faça upload do arquivo CSV para começar")

# Sidebar
st.sidebar.header("💡 Sobre")
st.sidebar.write("**Base Legal:** Art. 68 do Código Penal - Fases: 1.Pena base 2.Atenuantes/Agravantes 3.Majorantes/Minorantes 4.Cálculo 5.Regime 6.Substituição")
st.sidebar.write(f"**📊 Crimes carregados:** {len(crimes_data)}")

# Busca na sidebar
st.sidebar.write("**🔍 Buscar crime:**")
busca = st.sidebar.text_input("Digite o artigo ou descrição:")

if busca and crimes_data:
    crimes_filtrados = {k: v for k, v in crimes_data.items() if busca.lower() in k.lower()}
    st.sidebar.write(f"**Resultados ({len(crimes_filtrados)}):**")
    for chave in list(crimes_filtrados.keys())[:5]:
        crime_info = crimes_filtrados[chave]
        st.sidebar.write(f"**{crime_info['artigo']}** - Pena: {crime_info['pena_min']:.1f}-{crime_info['pena_max']:.1f} anos")

# Se não há dados carregados, mostrar mensagem
if not crimes_data:
    st.warning("""
    **⚠️ Aguardando upload do dataset**
    
    Para usar o simulador:
    1. **Faça upload do arquivo `crimes_cp_final_sem_art68.csv` acima**
    2. **Ou certifique-se que o arquivo está no repositório GitHub**
    
    O arquivo CSV deve conter as colunas:
    - Artigo_Base, Artigo_Completo, Descricao_Crime
    - Pena_Minima_Valor, Pena_Minima_Unidade
    - Pena_Maxima_Valor, Pena_Maxima_Unidade
    - Tipo_Penal_Estrutural
    """)
    st.stop()

# Fase 1: Pena Base e Circunstâncias
st.header("1️⃣ Fase 1: Pena Base e Circunstâncias")
col1, col2 = st.columns([2, 1])

with col1:
    if crimes_data:
        crime_selecionado = st.selectbox("Selecione o Crime:", options=list(crimes_data.keys()), format_func=lambda x: x)
        crime_info = crimes_data[crime_selecionado]
        min_pena = crime_info['pena_min']
        max_pena = crime_info['pena_max']
        
        st.write(f"**Artigo:** {crime_info['artigo']}")
        st.write(f"**Tipo penal:** {crime_info['tipo_penal']}")
        st.write(f"**Descrição:** {crime_info['descricao_completa']}")
        st.write(f"**Pena original:** {crime_info['pena_min_original']} {crime_info['unidade_original']} a {crime_info['pena_max_original']} {crime_info['unidade_original']}")
    else:
        st.error("Erro ao carregar dados dos crimes.")

with col2:
    circunstancia = st.radio("Circunstância do Crime:", ["Neutra", "Desfavorável", "Gravemente Desfavorável"])
    pena_base_inicial = min_pena
    ajuste_circunstancia = {"Neutra": 0, "Desfavorável": 0.2, "Gravemente Desfavorável": 0.4}
    fator_circunstancia = ajuste_circunstancia[circunstancia]
    pena_base_ajustada = pena_base_inicial * (1 + fator_circunstancia)
    
    st.write(f"**Pena prevista:** {min_pena:.1f} a {max_pena:.1f} anos")
    st.write(f"**Pena base inicial:** {pena_base_inicial:.1f} anos")
    st.write(f"**Circunstância {circunstancia.lower()}:** {fator_circunstancia*100:.0f}%")
    st.success(f"**PENA BASE APÓS CIRCUNSTÂNCIAS: {pena_base_ajustada:.1f} anos**")

# Fase 2: Atenuantes e Agravantes
st.header("2️⃣ Fase 2: Atenuantes e Agravantes Gerais")
col1, col2 = st.columns(2)

with col1:
    st.subheader("🔽 Atenuantes (Art. 65 CP)")
    atenuantes = st.multiselect("Selecione as atenuantes:", [
        "Réu primário de bons antecedentes", "Arrependimento espontâneo", 
        "Confissão espontânea", "Reparação do dano", "Coação moral", 
        "Embriaguez acidental", "Motivo de relevante valor social/moral"
    ])

with col2:
    st.subheader("🔼 Agravantes (Art. 61 CP)")
    agravantes = st.multiselect("Selecione as agravantes:", [
        "Reincidente específico", "Motivo fútil/torpe", "Crime contra idoso/doente", 
        "Uso de disfarce/emboscada", "Abuso de confiança/poder", 
        "Racismo/xenofobia", "Aumento do dano maliciosamente"
    ])

# Fase 3: Majorantes e Minorantes
st.header("3️⃣ Fase 3: Causas de Aumento/Diminuição")
majorantes_minorantes_generico = {
    "majorantes": [
        "Uso de arma (1/6 a 1/2)", "Violência grave (1/3 a 2/3)", 
        "Concurso de 2+ pessoas (1/4 a 1/2)", "Restrição à liberdade (1/6 a 1/3)", 
        "Abuso de confiança (1/6 a 1/3)"
    ],
    "minorantes": [
        "Valor ínfimo (1/6 a 1/3)", "Arrependimento posterior (1/6 a 1/3)", 
        "Circunstâncias atenuantes não previstas (1/6 a 1/3)"
    ]
}

col1, col2 = st.columns(2)
with col1:
    majorantes = st.multiselect("Causas de aumento (majorantes):", majorantes_minorantes_generico["majorantes"])
with col2:
    minorantes = st.multiselect("Causas de diminuição (minorantes):", majorantes_minorantes_generico["minorantes"])

# Fase 4: Cálculo Final
st.header("4️⃣ Fase 4: Cálculo Final da Pena")

if st.button("🎯 Calcular Pena Definitiva", type="primary"):
    pena_calculada = pena_base_ajustada
    
    st.subheader("📊 Detalhamento do Cálculo")
    calculo_detalhado = f"| Etapa | Valor | Ajuste |\n|-------|-------|---------|\n| **Pena Base Inicial** | {pena_base_inicial:.1f} anos | - |\n| Circunstância {circunstancia} | {pena_base_ajustada:.1f} anos | {fator_circunstancia*100:+.0f}% |\n"
    
    # Aplicar atenuantes
    for i, atenuante in enumerate(atenuantes, 1):
        reducao = pena_base_ajustada * (1/6)
        pena_calculada -= reducao
        calculo_detalhado += f"| Atenuante {i} | {pena_calculada:.1f} anos | -{reducao:.1f} anos |\n"
    
    # Aplicar agravantes
    for i, agravante in enumerate(agravantes, 1):
        aumento = pena_base_ajustada * (1/6)
        pena_calculada += aumento
        calculo_detalhado += f"| Agravante {i} | {pena_calculada:.1f} anos | +{aumento:.1f} anos |\n"
    
    # Aplicar majorantes
    for i, majorante in enumerate(majorantes, 1):
        aumento = pena_base_ajustada * (1/4)
        pena_calculada += aumento
        calculo_detalhado += f"| Majorante {i} | {pena_calculada:.1f} anos | +{aumento:.1f} anos |\n"
    
    # Aplicar minorantes
    for i, minorante in enumerate(minorantes, 1):
        reducao = pena_base_ajustada * (1/4)
        pena_calculada -= reducao
        calculo_detalhado += f"| Minorante {i} | {pena_calculada:.1f} anos | -{reducao:.1f} anos |\n"
    
    # Aplicar limites legais
    pena_final = max(min_pena, min(max_pena, pena_calculada))
    calculo_detalhado += f"| **LIMITES LEGAIS** | **{pena_final:.1f} anos** | **Ajuste final** |"
    
    st.markdown(calculo_detalhado)

    # Fase 5: Regime de Cumprimento
    st.header("5️⃣ Fase 5: Regime de Cumprimento")
    
    if pena_final > 8:
        regime = "FECHADO"
        cor_regime = "#ff4444"
        descricao = "Presídio de segurança máxima/média"
    elif pena_final >= 4:
        regime = "SEMIABERTO"
        cor_regime = "#ffaa00"
        descricao = "Colônia agrícola, industrial ou similar"
    else:
        regime = "ABERTO"
        cor_regime = "#44cc44"
        descricao = "Casa de albergado, trabalho externo"
    
    st.markdown(f"""
    <div style="background-color: {cor_regime}20; padding: 20px; border-radius: 10px; border-left: 5px solid {cor_regime};">
        <h2 style="color: {cor_regime}; margin: 0;">🔒 REGIME {regime}</h2>
        <p style="margin: 10px 0 0 0; font-size: 16px;"><strong>{descricao}</strong></p>
    </div>
    """, unsafe_allow_html=True)

    # Fase 6: Substituição da Pena
    st.header("6️⃣ Fase 6: Substituição da Pena")
    
    if pena_final <= 4:
        substituicao = "**CABE SUBSTITUIÇÃO** por pena restritiva de direitos"
        cor_subst = "#44cc44"
        fundamento = "Art. 44 CP - Penas até 4 anos podem ser substituídas"
    else:
        substituicao = "**NÃO CABE SUBSTITUIÇÃO**"
        cor_subst = "#ff4444"
        fundamento = "Art. 44 CP - Penas superiores a 4 anos não podem ser substituídas"
    
    st.markdown(f"""
    <div style="background-color: {cor_subst}20; padding: 15px; border-radius: 10px; border-left: 5px solid {cor_subst};">
        <h3 style="color: {cor_subst}; margin: 0;">{substituicao}</h3>
        <p style="margin: 5px 0 0 0;">{fundamento}</p>
    </div>
    """, unsafe_allow_html=True)

    # Gráfico da Dosimetria - VERSÃO MELHORADA
    st.header("📊 Evolução da Dosimetria da Pena")
    
    # Calcular posições no gráfico
    faixa_total = max_pena - min_pena
    if faixa_total > 0:
        pos_base = ((pena_base_inicial - min_pena) / faixa_total) * 100
        pos_ajustada = ((pena_base_ajustada - min_pena) / faixa_total) * 100
        pos_final = ((pena_final - min_pena) / faixa_total) * 100
    else:
        pos_base = pos_ajustada = pos_final = 50

    # Gráfico melhorado
    html_grafico = f'''
    <div style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); padding: 30px; border-radius: 15px; margin: 20px 0; box-shadow: 0 10px 30px rgba(0,0,0,0.2);">
        <h4 style="text-align: center; margin-bottom: 30px; color: white; font-weight: 600;">Evolução da Dosimetria da Pena</h4>
        
        <!-- Barra de progresso principal -->
        <div style="position: relative; height: 80px; background: rgba(255,255,255,0.2); border-radius: 40px; border: 3px solid rgba(255,255,255,0.3); margin-bottom: 80px; overflow: hidden;">
            <!-- Faixas de regime -->
            <div style="position: absolute; left: 0%; width: 33.3%; height: 100%; background: linear-gradient(90deg, #4CAF50, #8BC34A); border-right: 2px dashed white;"></div>
            <div style="position: absolute; left: 33.3%; width: 33.3%; height: 100%; background: linear-gradient(90deg, #FFC107, #FF9800); border-right: 2px dashed white;"></div>
            <div style="position: absolute; left: 66.6%; width: 33.4%; height: 100%; background: linear-gradient(90deg, #F44336, #E91E63);"></div>
            
            <!-- Marcadores -->
            <div style="position: absolute; left: {pos_base}%; top: 0; bottom: 0; width: 6px; background: #2196F3; transform: translateX(-50%); border-radius: 3px; box-shadow: 0 0 10px rgba(33, 150, 243, 0.8);">
                <div style="position: absolute; top: -45px; left: 50%; transform: translateX(-50%); white-space: nowrap; background: #2196F3; padding: 8px 12px; border-radius: 20px; border: 2px solid white; font-size: 12px; font-weight: bold; color: white; box-shadow: 0 4px 12px rgba(0,0,0,0.3);">
                    ⚖️ Base: {pena_base_inicial:.1f} anos
                </div>
            </div>
            
            <div style="position: absolute; left: {pos_ajustada}%; top: 0; bottom: 0; width: 6px; background: #9C27B0; transform: translateX(-50%); border-radius: 3px; box-shadow: 0 0 10px rgba(156, 39, 176, 0.8);">
                <div style="position: absolute; top: -45px; left: 50%; transform: translateX(-50%); white-space: nowrap; background: #9C27B0; padding: 8px 12px; border-radius: 20px; border: 2px solid white; font-size: 12px; font-weight: bold; color: white; box-shadow: 0 4px 12px rgba(0,0,0,0.3);">
                    📈 Ajustada: {pena_base_ajustada:.1f} anos
                </div>
            </div>
            
            <div style="position: absolute; left: {pos_final}%; top: 0; bottom: 0; width: 8px; background: #FF5722; transform: translateX(-50%); border-radius: 4px; box-shadow: 0 0 15px rgba(255, 87, 34, 0.9);">
                <div style="position: absolute; bottom: -50px; left: 50%; transform: translateX(-50%); white-space: nowrap; background: #FF5722; padding: 10px 15px; border-radius: 25px; border: 2px solid white; font-size: 13px; font-weight: bold; color: white; box-shadow: 0 4px 15px rgba(0,0,0,0.3);">
                    🎯 Final: {pena_final:.1f} anos
                </div>
            </div>
        </div>
        
        <!-- Legenda dos regimes -->
        <div style="display: flex; justify-content: space-between; margin-top: 30px;">
            <div style="text-align: center; flex: 1; margin: 0 5px;">
                <div style="background: linear-gradient(135deg, #4CAF50, #66BB6A); padding: 12px; border-radius: 10px; border: 2px solid white; box-shadow: 0 4px 12px rgba(0,0,0,0.2);">
                    <div style="font-weight: bold; color: white; font-size: 14px;">🔓 ABERTO</div>
                    <div style="color: rgba(255,255,255,0.9); font-size: 11px;">Até 4 anos</div>
                </div>
            </div>
            <div style="text-align: center; flex: 1; margin: 0 5px;">
                <div style="background: linear-gradient(135deg, #FFC107, #FFB300); padding: 12px; border-radius: 10px; border: 2px solid white; box-shadow: 0 4px 12px rgba(0,0,0,0.2);">
                    <div style="font-weight: bold; color: white; font-size: 14px;">🔐 SEMIABERTO</div>
                    <div style="color: rgba(255,255,255,0.9); font-size: 11px;">4 a 8 anos</div>
                </div>
            </div>
            <div style="text-align: center; flex: 1; margin: 0 5px;">
                <div style="background: linear-gradient(135deg, #F44336, #EF5350); padding: 12px; border-radius: 10px; border: 2px solid white; box-shadow: 0 4px 12px rgba(0,0,0,0.2);">
                    <div style="font-weight: bold; color: white; font-size: 14px;">🔒 FECHADO</div>
                    <div style="color: rgba(255,255,255,0.9); font-size: 11px;">Acima de 8 anos</div>
                </div>
            </div>
        </div>
        
        <!-- Escala numérica -->
        <div style="display: flex; justify-content: space-between; margin-top: 20px; padding: 0 10px;">
            <div style="color: white; font-size: 12px; font-weight: 500;">{min_pena:.1f} anos</div>
            <div style="color: white; font-size: 12px; font-weight: 500;">{(min_pena + max_pena)/2:.1f} anos</div>
            <div style="color: white; font-size: 12px; font-weight: 500;">{max_pena:.1f} anos</div>
        </div>
    </div>
    '''
    
    st.markdown(html_grafico, unsafe_allow_html=True)
    
    # Resumo final estilizado
    st.markdown(f"""
    <div style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); padding: 25px; border-radius: 15px; margin: 20px 0; text-align: center; box-shadow: 0 8px 25px rgba(0,0,0,0.2);">
        <h3 style="color: white; margin: 0 0 15px 0; font-weight: 600;">🎯 RESUMO FINAL</h3>
        <div style="display: flex; justify-content: space-around; align-items: center; flex-wrap: wrap;">
            <div style="background: rgba(255,255,255,0.9); padding: 15px; border-radius: 10px; margin: 5px; min-width: 200px;">
                <div style="font-weight: bold; color: #333; font-size: 16px;">Pena Final</div>
                <div style="font-size: 24px; font-weight: bold; color: #2196F3;">{pena_final:.1f} anos</div>
            </div>
            <div style="background: rgba(255,255,255,0.9); padding: 15px; border-radius: 10px; margin: 5px; min-width: 200px;">
                <div style="font-weight: bold; color: #333; font-size: 16px;">Regime</div>
                <div style="font-size: 20px; font-weight: bold; color: {cor_regime};">{regime}</div>
            </div>
            <div style="background: rgba(255,255,255,0.9); padding: 15px; border-radius: 10px; margin: 5px; min-width: 200px;">
                <div style="font-weight: bold; color: #333; font-size: 16px;">Substituição</div>
                <div style="font-size: 16px; font-weight: bold; color: {cor_subst};">{substituicao.replace('**', '')}</div>
            </div>
        </div>
    </div>
    """, unsafe_allow_html=True)

# Tabelas de Referência
st.header("📋 Tabela de Referência")
col1, col2, col3 = st.columns(3)

with col1:
    st.subheader("📊 Regimes")
    st.table(pd.DataFrame([
        {"Pena": "Até 4 anos", "Regime": "Aberto"},
        {"Pena": "4 a 8 anos", "Regime": "Semiaberto"},
        {"Pena": "Acima de 8 anos", "Regime": "Fechado"}
    ]))

with col2:
    st.subheader("⚖️ Fatores")
    st.table(pd.DataFrame([
        {"Fator": "Atenuante", "Ajuste": "-1/6"},
        {"Fator": "Agravante", "Ajuste": "+1/6"},
        {"Fator": "Majorante", "Ajuste": "+1/6 a +1/2"},
        {"Fator": "Minorante", "Ajuste": "-1/6 a -1/3"}
    ]))

with col3:
    st.subheader("🔀 Substituição")
    st.table(pd.DataFrame([
        {"Condição": "Pena ≤ 4 anos", "Substitui": "Sim"},
        {"Condição": "Pena > 4 anos", "Substitui": "Não"},
        {"Condição": "Réu reincidente", "Substitui": "Restrita"}
    ]))

st.markdown("---")
st.write("**⚖️ Ferramenta educacional - Consulte sempre a legislação atual e um profissional do direito**")
st.write("**📚 Base legal:** Arts. 59, 61, 65, 68 do Código Penal Brasileiro")
