import streamlit as st
import requests
import pandas as pd

st.set_page_config(page_title="Consulta de MeterId Azure", layout="centered")

st.title("🔍 Consulta de MeterId na Azure Retail API")
st.write("Digite um ou mais MeterIds (separados por vírgula) para consultar SKU, Serviço, Região e Moeda.")

# Campo de entrada
entrada = st.text_area("📥 Insira os MeterIds separados por vírgula", height=100)

# Regiões padrão para busca
regioes = ["brazilsouth", "eastus2", "Global", "Intercontinental", "Zone 1", "Zone 3"]

def consultar_meter_id(meter_id, regioes):
    for regiao in regioes:
        url = f"https://prices.azure.com/api/retail/prices?$filter=meterId eq '{meter_id}' and armRegionName eq '{regiao}'"
        response = requests.get(url)
        if response.status_code == 200:
            items = response.json().get("Items", [])
            if items:
                item = items[0]
                return {
                    "MeterId": meter_id,
                    "SKU_Name": item.get("skuName", "N/A"),
                    "Service_Name": item.get("serviceName", "N/A"),
                    "Azure_Region": item.get("armRegionName", "N/A"),
                    "Currency": item.get("currencyCode", "USD"),
                    "Unit_Price": item.get("unitPrice", 0.0)
                }
    return {
        "MeterId": meter_id,
        "SKU_Name": "❌ Não encontrado",
        "Service_Name": "-",
        "Azure_Region": "-",
        "Currency": "-",
        "Unit_Price": "-"
    }

if st.button("🔎 Consultar"):
    if entrada.strip():
        meter_ids = [m.strip() for m in entrada.split(",") if m.strip()]
        resultados = [consultar_meter_id(meter_id, regioes) for meter_id in meter_ids]
        df_resultado = pd.DataFrame(resultados)
        st.dataframe(df_resultado)

        # Botão para download
        csv = df_resultado.to_csv(index=False).encode("utf-8")
        st.download_button(
            label="📥 Baixar resultados em CSV",
            data=csv,
            file_name="consulta_meter_ids.csv",
            mime="text/csv"
        )
    else:
        st.warning("⚠️ Por favor, insira pelo menos um MeterId.")
