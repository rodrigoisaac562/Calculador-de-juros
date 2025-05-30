from shiny import App, render, ui
import math

def composto(capital, juros, tempo):
    if capital is None or juros is None or tempo is None:
        return None
    try:
        return capital * math.pow((1 + juros), tempo)
    except (TypeError, ValueError):
        return None

def simple(capital, juros, tempo):
    if capital is None or juros is None or tempo is None:
        return None
    try:
        juros_simples = capital * juros * tempo
        return capital + juros_simples
    except (TypeError, ValueError):
        return None

app_ui = ui.page_fluid(
    ui.panel_title("Calculadora de Juros"),
    ui.layout_sidebar(
        ui.sidebar(
            ui.input_radio_buttons(
                "metodo", "Método de Cálculo:",
                choices={"simples": "Simples", "composto": "Composto"},
                selected="composto"
            ),
            ui.input_numeric("capital", "Capital Inicial (R$)", value=1000, min=0),
            ui.input_numeric("juros", "Taxa de Juros Anual (%)", value=5, min=0),
            ui.input_numeric("tempo", "Tempo (meses)", value=12, min=1, step=1),
            width=3
        ),
        ui.h4("Resultado:"),
        ui.output_text_verbatim("resultado_calculo", placeholder=True)
    )
)

def server(input, output, session):
    @output
    @render.text
    def resultado_calculo():
        capital = input.capital()
        juros_percentual = input.juros()
        tempo_meses = input.tempo()
        metodo = input.metodo()

        if capital is None or juros_percentual is None or tempo_meses is None or metodo is None:
            return "Por favor, preencha todos os campos."

        if capital < 0 or juros_percentual < 0 or tempo_meses < 1:
             return "Por favor, insira valores válidos (Capital >= 0, Juros >= 0, Tempo >= 1)."

        try:
            juros_decimal = float(juros_percentual) / 100.0
            tempo_anos = int(tempo_meses) / 12.0
            capital_float = float(capital)

            resultado_str = f"Calculando Juros {metodo.capitalize()}...\n"
            resultado_str += f"Capital: R$ {capital_float:.2f}\n"
            resultado_str += f"Taxa Anual: {juros_percentual}%\n"
            resultado_str += f"Tempo: {tempo_meses} meses ({tempo_anos:.2f} anos)\n\n"

            if metodo == "composto":
                valor_final = composto(capital_float, juros_decimal, tempo_anos)
                if valor_final is not None:
                    juros_ganhos = valor_final - capital_float
                    resultado_str += f"Montante Final (Composto): R$ {valor_final:.2f}\n"
                    resultado_str += f"Total de Juros: R$ {juros_ganhos:.2f}"
                else:
                    resultado_str += "Erro ao calcular juros compostos."
            elif metodo == "simples":
                valor_final = simple(capital_float, juros_decimal, tempo_anos)
                if valor_final is not None:
                    juros_ganhos = valor_final - capital_float
                    resultado_str += f"Montante Final (Simples): R$ {valor_final:.2f}\n"
                    resultado_str += f"Total de Juros: R$ {juros_ganhos:.2f}"
                else:
                    resultado_str += "Erro ao calcular juros simples."
            else:
                resultado_str += "Método de cálculo inválido selecionado."

            return resultado_str

        except Exception as e:
            return f"Ocorreu um erro: {e}"

app = App(app_ui, server)
