import streamlit as st
import pandas as pd
from datetime import datetime, date
import sqlite3
import plotly.express as px

# Configuração da página
st.set_page_config(page_title="💰 Controle de Dívidas", layout="wide")
st.title("💰 Meu Controle de Dívidas")

# Conexão com o banco
conn = sqlite3.connect('dividas.db', check_same_thread=False)

# Criar tabelas
def init_db():
    conn.execute('''
    CREATE TABLE IF NOT EXISTS dividas (
        id INTEGER PRIMARY KEY,
        nome TEXT NOT NULL,
        credor TEXT,
        valor_total REAL,
        valor_atual REAL,
        juros_mensal REAL,
        data_vencimento TEXT,
        parcela_minima REAL,
        status TEXT DEFAULT 'Ativa'
    )''')
    
    conn.execute('''
    CREATE TABLE IF NOT EXISTS pagamentos (
        id INTEGER PRIMARY KEY,
        divida_id INTEGER,
        data TEXT,
        valor REAL,
        observacao TEXT
    )''')

init_db()

# Sidebar
menu = st.sidebar.selectbox("Menu", ["Dashboard", "Cadastrar Dívida", "Registrar Pagamento", "Minhas Dívidas", "Simulador"])

# ====================== CADASTRAR DÍVIDA ======================
if menu == "Cadastrar Dívida":
    st.subheader("Nova Dívida")
    with st.form("nova_divida"):
        nome = st.text_input("Nome da Dívida *")
        credor = st.text_input("Credor (ex: Nubank, Itaú)")
        valor_total = st.number_input("Valor Total Original", min_value=0.0, format="%.2f")
        valor_atual = st.number_input("Valor Atual (devido hoje)", min_value=0.0, format="%.2f")
        juros = st.number_input("Juros Mensal (%)", min_value=0.0, value=2.9, format="%.2f")
        vencimento = st.date_input("Próximo Vencimento", date.today())
        parcela = st.number_input("Parcela Mínima", min_value=0.0, format="%.2f")
        
        if st.form_submit_button("Salvar Dívida"):
            if nome and valor_atual > 0:
                conn.execute('''
                INSERT INTO dividas (nome, credor, valor_total, valor_atual, juros_mensal, data_vencimento, parcela_minima)
                VALUES (?, ?, ?, ?, ?, ?, ?)
                ''', (nome, credor, valor_total or valor_atual, valor_atual, juros, str(vencimento), parcela))
                conn.commit()
                st.success("Dívida cadastrada com sucesso!")
            else:
                st.error("Nome e Valor Atual são obrigatórios")

# ====================== DASHBOARD ======================
elif menu == "Dashboard":
    st.subheader("Dashboard Financeiro")
    
    df = pd.read_sql("SELECT * FROM dividas WHERE status = 'Ativa'", conn)
    
    if not df.empty:
        total_devido = df['valor_atual'].sum()
        qtd_dividas = len(df)
        
        col1, col2, col3 = st.columns(3)
        col1.metric("Total Devido", f"R$ {total_devido:,.2f}")
        col2.metric("Quantidade de Dívidas", qtd_dividas)
        col3.metric("Juros Médio", f"{df['juros_mensal'].mean():.2f}%")
        
        # Gráfico
        fig = px.bar(df, x='nome', y='valor_atual', title="Valor por Dívida", color='juros_mensal')
        st.plotly_chart(fig, use_container_width=True)
        
        st.dataframe(df[['nome', 'credor', 'valor_atual', 'juros_mensal', 'data_vencimento']], use_container_width=True)
    else:
        st.info("Nenhuma dívida cadastrada ainda.")

# ====================== REGISTRAR PAGAMENTO ======================
elif menu == "Registrar Pagamento":
    st.subheader("Registrar Pagamento")
    dividas = pd.read_sql("SELECT id, nome, valor_atual FROM dividas WHERE status = 'Ativa'", conn)
    
    if not dividas.empty:
        divida_selecionada = st.selectbox("Qual dívida?", dividas['nome'])
        divida_id = dividas[dividas['nome'] == divida_selecionada]['id'].iloc[0]
        
        valor_pago = st.number_input("Valor a pagar", min_value=0.0, format="%.2f")
        obs = st.text_input("Observação (opcional)")
        
        if st.button("Registrar Pagamento"):
            if valor_pago > 0:
                # Registrar pagamento
                conn.execute("INSERT INTO pagamentos (divida_id, data, valor, observacao) VALUES (?, ?, ?, ?)",
                           (divida_id, str(date.today()), valor_pago, obs))
                
                # Atualizar valor da dívida
                conn.execute("UPDATE dividas SET valor_atual = valor_atual - ? WHERE id = ?", (valor_pago, divida_id))
                
                # Verificar se quitou
                novo_valor = pd.read_sql(f"SELECT valor_atual FROM dividas WHERE id = {divida_id}", conn)['valor_atual'].iloc[0]
                if novo_valor <= 0:
                    conn.execute("UPDATE dividas SET status = 'Quitada' WHERE id = ?", (divida_id,))
                
                conn.commit()
                st.success(f"Pagamento de R$ {valor_pago:.2f} registrado!")
    else:
        st.warning("Cadastre dívidas primeiro.")

# ====================== MINHAS DÍVIDAS ======================
elif menu == "Minhas Dívidas":
    st.subheader("Todas as Dívidas")
    df = pd.read_sql("SELECT * FROM dividas", conn)
    if not df.empty:
        st.dataframe(df, use_container_width=True)
    else:
        st.info("Ainda não há dívidas cadastradas.")

# ====================== SIMULADOR ======================
elif menu == "Simulador":
    st.subheader("Simulador de Quitação")
    df = pd.read_sql("SELECT * FROM dividas WHERE status = 'Ativa'", conn)
    
    if not df.empty:
        valor_extra = st.number_input("Quanto você pode pagar a mais por mês?", min_value=0.0, value=500.0, format="%.2f")
        st.write("Em breve vou melhorar bastante esse simulador com projeções mês a mês.")
        
        total = df['valor_atual'].sum()
        st.metric("Tempo estimado aproximado", f"{total / (df['parcela_minima'].sum() + valor_extra):.1f} meses")
    else:
        st.info("Cadastre dívidas para usar o simulador.")

st.caption("App de Controle de Dívidas - Versão 1.0")

streamlit run controle_dividas.py
