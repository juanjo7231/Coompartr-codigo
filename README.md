import streamlit as st
import sympy as sp
from sympy.calculus.util import continuous_domain
from sympy.parsing.sympy_parser import (parse_expr, standard_transformations,
                                        implicit_multiplication_application, convert_xor)
import re
import os
import json
import html

# --- ESTILOS CSS GENERALES ---
st.markdown("""
    <style>
    @import url('https://fonts.googleapis.com/css2?family=Lora:ital,wght@0,500;1,500&display=swap');

    [data-testid="manage-app-button"] {
        display: none !important;
    }
    
    .viewerBadge_container__1QSob {
        display: none !important;
    }
    
    @keyframes pulsoEntradaVerde {
        0% { box-shadow: 0 0 0 0 rgba(0, 122, 51, 0.7); }
        70% { box-shadow: 0 0 0 14px rgba(0, 122, 51, 0); }
        100% { box-shadow: 0 0 0 0 rgba(0, 122, 51, 0); }
    }

    @keyframes fadeInApp {
        from { opacity: 0; transform: translateY(6px); }
        to { opacity: 1; transform: translateY(0); }
    }

    .stApp {
        background-color: #F8FAFC;
        color: #1E293B !important;
        animation: fadeInApp 0.6s ease-out;
    }
    
    .stChatMessage {
        color: #1E293B !important;
    }
    .stChatMessage p, .stChatMessage span, .stChatMessage div {
        color: #1E293B !important;
    }
    
    .stChatInput {
        width: min(860px, calc(100vw - 80px)) !important;
        max-width: 860px !important;
        animation: pulsoEntradaVerde 2.5s infinite;
        border-radius: 10px;
    }

    .stChatInput textarea, .stChatInput input {
        color: #FFFFFF !important;
        background-color: #1E293B !important;
        -webkit-text-fill-color: #FFFFFF !important;
    }
    div[data-baseweb="input"], div[data-baseweb="base-input"], div[data-baseweb="textarea"] {
        background-color: #1E293B !important;
        border-radius: 8px !important;
    }
    .stChatInput textarea::placeholder {
        color: #94A3B8 !important;
        opacity: 1 !important;
    }

    .uis-header {
        background-color: #007A33;
        color: white;
        padding: 16px 20px;
        border-radius: 12px;
        display: flex;
        align-items: center;
        justify-content: space-between;
        margin-bottom: 20px;
        box-shadow: 0 4px 12px rgba(0,0,0,0.04);
    }
    
    .tip-box-chat {
        position: relative;
        background: linear-gradient(135deg, #F4FCF7 0%, #E7F7EE 100%);
        border: 1px solid #C7E7D3;
        border-left: 5px solid #007A33;
        padding: 15px 18px 15px 18px;
        border-radius: 10px;
        margin: 12px 0;
        color: #1E293B !important;
        font-size: 14px;
        line-height: 1.6;
        box-shadow: 0 3px 10px rgba(0, 94, 39, 0.07);
        transition: transform 0.2s ease, box-shadow 0.2s ease;
    }

    .tip-box-chat:hover {
        transform: translateY(-1px);
        box-shadow: 0 6px 16px rgba(0, 94, 39, 0.11);
    }

    .tip-box-chat strong:first-of-type {
        color: #005E27 !important;
        font-weight: 750 !important;
    }

    .tip-box-chat::before {
        content: "";
        position: absolute;
        top: 13px;
        right: 14px;
        width: 7px;
        height: 7px;
        border-radius: 50%;
        background: #65B741;
        opacity: 0.65;
    }

    .tips-intro {
        background: #FFFFFF;
        border: 1px solid #DCE8E1;
        border-radius: 11px;
        padding: 14px 17px;
        margin: 3px 0 14px 0;
        box-shadow: 0 2px 8px rgba(15, 23, 42, 0.035);
        color: #334155 !important;
        font-size: 14px;
        line-height: 1.55;
    }

    .tips-intro-title {
        color: #005E27 !important;
        font-size: 15px;
        font-weight: 750;
        margin-bottom: 3px;
    }

    .solution-intro {
        background: linear-gradient(135deg, #F8FCFA 0%, #EDF8F1 100%);
        border: 1px solid #CFE6D8;
        border-top: 4px solid #007A33;
        border-radius: 12px;
        padding: 16px 18px;
        margin: 6px 0 16px 0;
        box-shadow: 0 4px 13px rgba(0, 94, 39, 0.07);
    }

    .solution-title {
        color: #005E27 !important;
        font-size: 18px;
        font-weight: 750;
        line-height: 1.35;
        margin: 0 0 5px 0;
    }

    .solution-subtitle {
        color: #475569 !important;
        font-size: 13.5px;
        line-height: 1.55;
        margin: 0;
    }

    .step-divider {
        height: 1px;
        background: linear-gradient(90deg, transparent, #C9D9D0, transparent);
        margin: 16px 0 5px 0;
    }

    .aviso-box-chat {
        background-color: #FEF3E2;
        border-left: 4px solid #D97706;
        padding: 14px 16px;
        border-radius: 0 8px 8px 0;
        margin: 10px 0;
        color: #1E293B !important;
        font-size: 14px;
        line-height: 1.5;
    }

    .paso-header {
        color: #005E27 !important;
        font-weight: 750;
        font-size: 16px;
        line-height: 1.35;
        margin: 22px 0 8px 0;
        padding: 9px 12px;
        background: #F3FAF6;
        border: 1px solid #D4E8DC;
        border-left: 4px solid #007A33;
        border-radius: 8px;
        box-shadow: 0 2px 7px rgba(0, 94, 39, 0.045);
    }
    .paso-cuerpo {
        color: #334155 !important;
        font-size: 14.5px;
        line-height: 1.68;
        margin: 5px 4px 8px 4px;
        padding: 0 7px;
    }
    .paso-subtext {
        color: #64748B !important;
        font-size: 13px;
        line-height: 1.55;
        margin: 4px 4px 9px 4px;
        padding: 7px 10px;
        background: #F8FAFC;
        border-left: 3px solid #CBD5E1;
        border-radius: 0 7px 7px 0;
        font-style: italic;
    }

    .stLatex {
        margin: 8px 0 !important;
    }

    .solution-intro + div {
        margin-top: 0 !important;
    }

    .mirror-line {
        font-family: 'Consolas', 'Menlo', monospace;
        font-size: 13.5px;
        font-weight: 700;
        color: #005E27 !important;
        background-color: #E2F6EC;
        padding: 6px 10px;
        border-radius: 6px;
        display: inline-block;
        margin-bottom: 10px;
    }
    
    .stButton > button {
        background: linear-gradient(135deg, #008738 0%, #005E27 100%) !important;
        color: #FFFFFF !important;
        border-radius: 10px !important;
        border: 1px solid rgba(255, 255, 255, 0.15) !important;
        font-weight: 600 !important;
        padding: 0.6rem 1.2rem !important;
        box-shadow: 0 4px 10px rgba(0, 122, 51, 0.25) !important;
        transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1) !important;
        width: 100% !important;
        letter-spacing: 0.3px;
    }
    
    .stButton > button p, .stButton > button span, .stButton > button div {
        color: #FFFFFF !important;
    }

    .stButton > button:hover {
        background: linear-gradient(135deg, #00A642 0%, #007A33 100%) !important;
        box-shadow: 0 6px 16px rgba(0, 122, 51, 0.4) !important;
        transform: translateY(-2px) !important;
        border-color: rgba(255, 255, 255, 0.3) !important;
    }

    [data-testid="stChatMessageContent"] [data-testid="stVerticalBlock"] {
        gap: 0 !important;
    }

    [data-testid="stChatMessageContent"] [data-testid="stElementContainer"] {
        margin-top: 0 !important;
        margin-bottom: 0 !important;
        padding-top: 0 !important;
        padding-bottom: 0 !important;
    }

    [data-testid="stChatMessageContent"] .paso-header {
        margin: 24px 0 8px 0 !important;
        padding: 10px 13px !important;
    }

    [data-testid="stChatMessageContent"] .step-divider {
        margin: 14px 0 4px 0 !important;
        height: 1px !important;
    }

    [data-testid="stChatMessageContent"] .paso-cuerpo {
        margin: 4px 0 5px 0 !important;
        padding: 0 7px !important;
        line-height: 1.6 !important;
    }

    [data-testid="stChatMessageContent"] .paso-subtext {
        margin: 3px 0 6px 0 !important;
        padding: 6px 10px !important;
        line-height: 1.45 !important;
    }

    [data-testid="stChatMessageContent"] [data-testid="stElementContainer"] .stLatex,
    [data-testid="stChatMessageContent"] [data-testid="stElementContainer"] .katex-display {
        margin: 3px 0 5px 0 !important;
        padding: 2px 0 !important;
    }

    [data-testid="stChatMessageContent"] .step-divider {
        margin: 7px 0 3px 0 !important;
    }

    [data-testid="stChatMessageContent"] .paso-header + div {
        margin-top: 0 !important;
    }

    [data-testid="stChatMessageContent"] [data-testid="stVerticalBlock"] { gap: 0.35rem !important; }
    [data-testid="stChatMessageContent"] [data-testid="stElementContainer"] { margin-bottom: 0 !important; }
    [data-testid="stChatMessageContent"] .paso-header { margin-top: 14px !important; margin-bottom: 5px !important; }
    [data-testid="stChatMessageContent"] .paso-cuerpo { margin-top: 2px !important; margin-bottom: 3px !important; }
    [data-testid="stChatMessageContent"] .paso-subtext { margin-top: 2px !important; margin-bottom: 5px !important; }
    [data-testid="stChatMessageContent"] .stLatex { margin: 2px 0 !important; }

    .stChatMessage .paso-header {
        background: linear-gradient(90deg, #EEF8F2 0%, #FFFFFF 100%);
        border: 1px solid #D7E6DC;
        border-left: 5px solid #007A33;
        border-radius: 9px;
        padding: 10px 14px;
        margin: 26px 0 12px 0;
        box-shadow: 0 2px 8px rgba(15, 23, 42, 0.045);
        color: #005E27 !important;
        font-size: 16px;
        font-weight: 700;
        line-height: 1.4;
    }

    .stChatMessage .paso-cuerpo {
        background: #FFFFFF;
        border-left: 2px solid #D7E6DC;
        padding: 5px 12px 6px 14px;
        margin: 0 0 8px 4px;
        color: #334155 !important;
        font-size: 14.5px;
        line-height: 1.7;
    }

    .stChatMessage .paso-subtext {
        background: #F8FAFC;
        border: 1px solid #E2E8F0;
        border-radius: 7px;
        padding: 8px 11px;
        margin: 8px 4px 14px 4px;
        color: #64748B !important;
        font-size: 13px;
        line-height: 1.55;
        font-style: normal;
    }

    .stChatMessage .katex-display {
        margin: 16px 8px 18px 8px !important;
        padding: 9px 8px !important;
        overflow-x: auto;
        overflow-y: hidden;
    }

    .procedimiento-titulo {
        background: #FFFFFF;
        border: 1px solid #D7E6DC;
        border-radius: 10px;
        padding: 12px 15px;
        margin: 18px 0 10px 0;
        color: #005E27 !important;
        font-size: 17px;
        font-weight: 700;
        box-shadow: 0 2px 8px rgba(15, 23, 42, 0.04);
    }

    .procedimiento-intro {
        color: #475569 !important;
        font-size: 14px;
        line-height: 1.6;
        margin: 0 0 8px 2px;
    }

    section[data-testid="stSidebar"] div[data-testid="stButton"] > button,
    section[data-testid="stSidebar"] div[data-testid="stButton"] > button p,
    section[data-testid="stSidebar"] div[data-testid="stButton"] > button span,
    section[data-testid="stSidebar"] div[data-testid="stButton"] > button div {
        font-family: inherit !important;
        font-size: 14px !important;
        font-style: normal !important;
        font-weight: 400 !important;
        line-height: 1.45 !important;
        letter-spacing: 0 !important;
        text-align: left !important;
    }

    [data-testid="stMarkdownContainer"] table {
        width: 100% !important;
        border-collapse: collapse !important;
        border: 2px solid #111827 !important;
        border-radius: 8px !important;
        overflow: hidden !important;
        margin: 14px 0 18px 0 !important;
        background: #FFFFFF !important;
        font-size: 14px !important;
    }

    [data-testid="stMarkdownContainer"] table thead th {
        background: #E5E7EB !important;
        color: #111827 !important;
        border: 1px solid #111827 !important;
        padding: 11px 13px !important;
        font-weight: 700 !important;
        text-align: left !important;
    }

    [data-testid="stMarkdownContainer"] table tbody td {
        color: #1E293B !important;
        border: 1px solid #111827 !important;
        padding: 10px 13px !important;
        vertical-align: middle !important;
        background: #FFFFFF !important;
    }

    [data-testid="stMarkdownContainer"] table tbody tr:nth-child(even) td {
        background: #F8FAFC !important;
    }

    [data-testid="stMarkdownContainer"] table tbody td:first-child {
        font-weight: 600 !important;
        white-space: nowrap !important;
    }

    .welcome-box {
        background: #FFFFFF;
        border: 1px solid #D7E2DC;
        border-left: 5px solid #007A33;
        border-radius: 10px;
        padding: 18px 20px;
        margin: 4px 0 14px 0;
        box-shadow: 0 3px 10px rgba(15, 23, 42, 0.04);
    }

    .welcome-title {
        color: #005E27 !important;
        font-size: 16px;
        font-weight: 700;
        line-height: 1.4;
        margin: 0 0 8px 0;
    }

    .welcome-text {
        color: #334155 !important;
        font-size: 14px;
        line-height: 1.6;
        margin: 0 0 8px 0;
    }

    .welcome-instruction {
        color: #1E293B !important;
        font-size: 14px;
        line-height: 1.55;
        margin: 0;
    }
    
    .logo-circular img {
        border-radius: 50%;
        border: 2px solid #007A33;
        object-fit: cover;
        width: 48px;
        height: 48px;
    }

    @media (max-width: 768px) {
        .uis-header {
            flex-direction: column;
            align-items: flex-start;
            gap: 8px;
            padding: 12px 14px;
        }
        .uis-header h3 {
            font-size: 16px !important;
        }
        .block-container {
            padding-left: 1rem !important;
            padding-right: 1rem !important;
            padding-top: 2rem !important;
        }
    }
    </style>
""", unsafe_allow_html=True)

x = sp.Symbol('x', real=True)


# ------------------------------------------------------------------
# SYSTEM PROMPTS (IA)
# ------------------------------------------------------------------
# Funciones f(x) dadas: las resuelve el motor algebraico propio (6 pasos).
# Problemas con enunciado: la IA plantea el modelo (SYSTEM_PROMPT_MODELADO),
# SymPy verifica la sustitución de la restricción, el motor resuelve con
# rigor y la IA redacta la conclusión (SYSTEM_PROMPT_REDACCION).
MODELO_CLAUDE = "claude-sonnet-5-5"

SYSTEM_PROMPT_MODELADO = """Eres un experto en Cálculo 1 que PLANTEA problemas de optimización con enunciado. NO resuelves el problema: solo lo modelas para que otro motor lo derive.

Responde ÚNICAMENTE con un objeto JSON válido (sin texto antes o después, sin bloques de código) con estas claves:
{
  "es_optimizacion": true o false,
  "mensaje": "si no es un problema de optimización de Cálculo 1, explica en una frase por qué; si lo es, cadena vacía",
  "planteamiento": "texto en Markdown con LaTeX ($...$) que explique, paso a paso y en lenguaje de principiante: (A) qué se quiere maximizar o minimizar y qué datos hay, (B) las variables con sus unidades, (C) la función objetivo y la restricción, (D) cómo se despeja para dejar UNA sola variable x, (E) el dominio con sentido físico. No derives ni resuelvas.",
  "funcion": "la función objetivo en UNA variable llamada x, escrita para Python/SymPy: usa ** para potencias, * para multiplicar, sqrt(), exp(), ln(), pi",
  "objetivo_expr": "la cantidad a optimizar con TODAS sus variables originales (ej: 2*x*y), sintaxis SymPy; x es la variable independiente",
  "restricciones": ["ecuaciones del enunciado como 'izquierda = derecha' (ej: 'x + 2*y = 100')"],
  "dominio": {"min": número o null, "max": número o null, "min_incluido": true o false, "max_incluido": true o false},
  "objetivo": "maximizar" o "minimizar",
  "descripcion_x": "qué representa x, con unidades",
  "nombre_objetivo": "nombre de la cantidad optimizada (por ejemplo: área, costo, volumen)",
  "unidad_objetivo": "unidades de la cantidad optimizada",
  "otras_variables": [{"nombre": "nombre de otra medida pedida", "expresion": "expresión en x para SymPy", "unidad": "unidades"}]
}

Reglas: la variable independiente SIEMPRE se llama x; las demás variables usan letras distintas (y, r, h...). La función debe quedar en una sola variable usando la restricción. El dominio debe reflejar lo físicamente posible (medidas positivas, límites del enunciado). Usa null en un límite que no esté acotado. Si el enunciado no da datos suficientes, inventa NINGÚN dato: marca es_optimizacion como false y explica qué falta. Prohibido usar sign, sgn o deltas de Dirac."""

SYSTEM_PROMPT_REDACCION = """Redactas la conclusión final de un problema de optimización aplicada de Cálculo 1, en español, para un estudiante principiante.

Recibirás el enunciado, el objetivo y una lista de HECHOS ya calculados. Reglas:
- Usa SOLO los números de los hechos; no calcules ni inventes valores nuevos.
- Escribe 2 a 4 oraciones, con unidades, respondiendo exactamente lo que pregunta el enunciado.
- Indica que el resultado corresponde al máximo o al mínimo pedido y que fue verificado en el análisis de los pasos anteriores.
- Prohibido sign, sgn, deltas, la palabra "fallback" y las "Fases". No hagas preguntas ni ofrezcas menús."""


def hacer_reales_potencias_fraccionarias(expr):
    """
    Corrige un problema de fondo de sympy: al evaluar numéricamente una
    potencia racional con base negativa (p. ej. x**(2/3) en x = -1), sympy
    usa por defecto la rama principal de la exponenciación compleja, que
    para exponentes con denominador impar NO coincide con la convención de
    análisis real.

    Para cada Pow(base, p/q) con q impar y exponente no entero se reescribe:
        base**(p/q) = |base|**(p/q)                si p es par
        base**(p/q) = sign(base) * |base|**(p/q)    si p es impar
    IMPORTANTE: SOLO para el motor interno. Nunca se muestra al estudiante.
    """
    reemplazos = {}
    for p in expr.atoms(sp.Pow):
        base, exp = p.as_base_exp()
        if exp.is_Rational and not exp.is_Integer and exp.q % 2 == 1:
            if exp.p % 2 == 0:
                reemplazos[p] = sp.Abs(base) ** exp
            else:
                reemplazos[p] = sp.sign(base) * sp.Abs(base) ** exp
    if reemplazos:
        expr = expr.xreplace(reemplazos)
    return expr


_PATRON_SIGN_LATEX = re.compile(
    r'\\operatorname\{sign\}\{?\\left\(([^)]*)\\right\)\}?|\\operatorname\{sign\}\(([^)]*)\)|\\mathrm\{sign\}\\left\(([^)]*)\\right\)'
)


def latex_sin_sign(expr) -> str:
    """
    Wrapper OBLIGATORIO alrededor de sp.latex(): ninguna salida mostrada al
    estudiante puede contener sign, sgn ni deltas de Dirac.
    """
    try:
        cadena = sp.latex(expr)
    except Exception:
        return str(expr).replace("DiracDelta", "").replace("sign", "").replace("sgn", "").replace("δ", "")

    cadena = re.sub(r'\\operatorname\{DiracDelta\}\s*\((.*?)\)', '', cadena)
    cadena = re.sub(r'\\operatorname\{sign\}\s*\((.*?)\)', '', cadena)
    cadena = re.sub(r'\\mathrm\{sign\}\s*\((.*?)\)', '', cadena)
    cadena = re.sub(r'\\operatorname\{sgn\}\s*\((.*?)\)', '', cadena)
    cadena = cadena.replace(r'\delta', '')

    def _reemplazo(m):
        interior = m.group(1) or m.group(2) or m.group(3) or "u"
        return (f"\\left[\\text{{+1 si }} {interior} \\geq 0 \\text{{; }} "
                f"-1 \\text{{ si }} {interior} < 0\\right]")

    return _PATRON_SIGN_LATEX.sub(_reemplazo, cadena)


# ------------------------------------------------------------------
# POST-PROCESAMIENTO DE SALIDA (red de seguridad con regex)
# ------------------------------------------------------------------
_PATRONES_TEXTO_PROHIBIDO = [
    (re.compile(r'fallback[^.\n]*[.]?', re.IGNORECASE), ''),
    (re.compile(r'restricciones\s+trascendentes[^.\n]*[.]?', re.IGNORECASE), ''),
    (re.compile(r'\bFase\s*\d+\s*[—\-:]\s*'), ''),
    (re.compile(r'DiracDelta\s*\([^)]*\)'), ''),
    (re.compile(r'\b(?:sign|sgn)\s*\([^)]*\)', re.IGNORECASE), ''),
    (re.compile(r'\bδ\s*\([^)]*\)'), ''),
]


def post_procesar_latex(cadena):
    """Limpia cualquier resto de objetos de motor simbólico en un LaTeX."""
    if not isinstance(cadena, str):
        return cadena
    t = cadena
    t = re.sub(r'\\operatorname\{DiracDelta\}\s*(?:\\left)?\((?:[^()]|\([^()]*\))*(?:\\right)?\)', '', t)
    t = re.sub(r'\\operatorname\{(?:sign|sgn)\}\s*(?:\\left)?\((?:[^()]|\([^()]*\))*(?:\\right)?\)', '', t)
    t = re.sub(r'\\mathrm\{(?:sign|sgn)\}\s*(?:\\left)?\((?:[^()]|\([^()]*\))*(?:\\right)?\)', '', t)
    t = t.replace(r'\delta', '')
    return t


def post_procesar_texto(texto):
    """Limpia texto/Markdown: sin sign, deltas, 'fallback' ni etiquetas de 'Fase'."""
    if not isinstance(texto, str):
        return texto
    t = texto
    for patron, repl in _PATRONES_TEXTO_PROHIBIDO:
        t = patron.sub(repl, t)
    t = re.sub(r'[ \t]{2,}', ' ', t)
    return t


FUNCIONES_CONOCIDAS = ['sqrt', 'asin', 'acos', 'atan', 'sinh', 'cosh', 'tanh',
                        'sin', 'cos', 'tan', 'cot', 'sec', 'csc', 'log', 'ln', 'exp',
                        'Abs', 'abs']


SUPERINDICES_A_NORMAL = {
    '⁰': '0', '¹': '1', '²': '2', '³': '3', '⁴': '4',
    '⁵': '5', '⁶': '6', '⁷': '7', '⁸': '8', '⁹': '9', '⁻': '-',
}


def detectar_posible_ambiguedad_fraccion(texto_original: str):
    """
    Detecta el error típico "x/x^2+1" pensando en x/(x^2+1).
    Devuelve (denominador_interpretado, signo, resto) o None.
    """
    patron = re.compile(
        r'/\s*([a-zA-Z_]\w*(?:\s*\^\s*\d+)?)\s*([+\-])\s*(.+)$'
    )
    m = patron.search(texto_original)
    if not m:
        return None
    denominador = m.group(1).strip()
    signo = m.group(2).strip()
    resto = m.group(3).strip()
    return denominador, signo, resto


def extraer_intervalo_usuario(texto: str):
    """
    Busca un intervalo cerrado explícito [a, b]. Devuelve
    (a, b, texto_sin_intervalo) o None.
    """
    patron = re.compile(r'\[\s*(-?\d+(?:\.\d+)?(?:/\d+)?)\s*,\s*(-?\d+(?:\.\d+)?(?:/\d+)?)\s*\]')
    m = patron.search(texto)
    if not m:
        return None
    try:
        a = sp.nsimplify(m.group(1))
        b = sp.nsimplify(m.group(2))
    except Exception:
        return None
    if a > b:
        a, b = b, a
    antes = texto[:m.start()]
    antes = re.sub(r'(?:\bpara\s+)?(?:\bx\s*)?(?:\ben\b|\bin\b|∈)\s*$', '', antes.rstrip(), flags=re.IGNORECASE)
    texto_sin_intervalo = (antes + texto[m.end():]).strip()
    return a, b, texto_sin_intervalo


def desempacar_intervalo(iv):
    """Acepta (a, b) cerrado o (a, b, izq_abierto, der_abierto)."""
    if len(iv) == 4:
        a, b, lo, ro = iv
    else:
        a, b = iv
        lo = ro = False
    return a, b, bool(lo), bool(ro)


def texto_intervalo_usuario(iv):
    a, b, lo, ro = desempacar_intervalo(iv)
    return _fmt_intervalo(a, b, not lo, not ro)


def es_funcion_pura(texto):
    """
    True si el texto es una expresión matemática en x y no una frase con
    palabras: decide si lo resuelve el motor algebraico o si es un problema
    con enunciado.
    """
    t = texto.strip()
    if not t:
        return False
    try:
        iv = extraer_intervalo_usuario(t)
        if iv is not None:
            t = iv[2]
        saneada = limpiar_sintaxis_matematica(t)
        palabras = re.findall(r'[A-Za-z_]+', saneada)
        permitidas = set(FUNCIONES_CONOCIDAS) | {'x', 'e', 'E', 'pi', 'oo'}
        if any(p not in permitidas for p in palabras):
            return False
        sp.sympify(saneada, locals={'x': x, 'ln': sp.log})
        return True
    except Exception:
        return False


class ErrorProblemaAplicado(Exception):
    """Mensaje amable para el estudiante cuando no se puede resolver un enunciado."""


def _cliente_anthropic():
    try:
        import anthropic
    except Exception:
        return None
    clave = None
    try:
        clave = st.secrets.get("ANTHROPIC_API_KEY")
    except Exception:
        clave = None
    clave = clave or os.environ.get("ANTHROPIC_API_KEY")
    if not clave:
        return None
    return anthropic.Anthropic(api_key=clave)


def llamar_claude(system, usuario, max_tokens=2000):
    """Llamada simple a la API de Anthropic. Lanza RuntimeError('SIN_API') si no hay clave."""
    cliente = _cliente_anthropic()
    if cliente is None:
        raise RuntimeError("SIN_API")
    r = cliente.messages.create(
        model=MODELO_CLAUDE,
        max_tokens=max_tokens,
        system=system,
        messages=[{"role": "user", "content": usuario}],
    )
    return "".join(b.text for b in r.content if getattr(b, "type", "") == "text")


def extraer_json(texto):
    t = texto.strip()
    t = re.sub(r'^```(?:json)?\s*|\s*```$', '', t)
    i, j = t.find('{'), t.rfind('}')
    if i == -1 or j == -1:
        raise ValueError("sin JSON")
    return json.loads(t[i:j + 1])


def encontrar_puntos_no_diferenciables(f_expr, f_prime, dominio):
    """
    Puntos donde f(x) SÍ está definida pero f'(x) podría NO existir:
    denominador de f' = 0, argumento de |.| = 0, base de potencia
    racional 0 < exp < 1 igual a 0.
    """
    candidatos = []

    def _agregar(valor):
        try:
            if sp.im(sp.N(valor)) != 0:
                return
            v_real = sp.re(valor)
        except Exception:
            return
        try:
            en_dominio = dominio is None or dominio.contains(v_real) != False
        except Exception:
            en_dominio = True
        if not en_dominio:
            return
        try:
            val_f_n = sp.N(f_expr.subs(x, v_real))
            if not val_f_n.is_finite:
                return
        except Exception:
            return
        candidatos.append(v_real)

    denom_fp = sp.denom(sp.together(f_prime))
    if denom_fp != 1:
        try:
            for c in sp.solve(sp.Eq(denom_fp, 0), x):
                _agregar(c)
        except Exception:
            pass

    for ab in f_expr.atoms(sp.Abs):
        try:
            for c in sp.solve(sp.Eq(ab.args[0], 0), x):
                _agregar(c)
        except Exception:
            pass

    for p in f_expr.atoms(sp.Pow):
        try:
            if p.exp.is_Rational and 0 < p.exp < 1:
                for c in sp.solve(sp.Eq(p.base, 0), x):
                    _agregar(c)
        except Exception:
            pass

    vistos = []
    unicos = []
    for c in candidatos:
        if c not in vistos:
            vistos.append(c)
            unicos.append(c)
    return unicos


def _condicion_de_rama_valida(u_expr, valor_x, es_positivo):
    """Comprueba si u_expr(valor_x) cumple el signo (>=0 o <0) que define la rama."""
    try:
        val = sp.N(u_expr.subs(x, valor_x))
        if val.is_real is False:
            return False
        val_f = float(val)
    except Exception:
        return False
    tolerancia = 1e-9
    if es_positivo:
        return val_f >= -tolerancia
    else:
        return val_f < tolerancia


def resolver_puntos_estacionarios(f_expr, dominio):
    """
    Puntos donde f'(x) = 0 (motor interno). Con |u(x)| se abren ramas
    (u >= 0 y u < 0) y una solución solo se acepta si cumple su rama y
    pertenece al dominio.
    """
    abs_atoms = list(f_expr.atoms(sp.Abs))
    sign_atoms = list(f_expr.atoms(sp.sign))

    if not abs_atoms:
        f_prime = sp.diff(f_expr, x)
        try:
            soluciones = sp.solve(sp.Eq(f_prime, 0), x)
        except Exception:
            soluciones = []
        puntos = []
        for s in soluciones:
            try:
                if sp.im(sp.N(s)) == 0:
                    s_real = sp.re(s)
                    if dominio is None or dominio.contains(s_real) != False:
                        puntos.append(s_real)
            except Exception:
                pass
        return puntos

    n = len(abs_atoms)
    puntos_totales = []
    vistos = []

    for mascara in range(2 ** n):
        reemplazos_rama = {}
        condiciones = []
        positivo_por_u = []
        for i, ab in enumerate(abs_atoms):
            u_expr = ab.args[0]
            es_positivo = bool((mascara >> i) & 1)
            reemplazos_rama[ab] = u_expr if es_positivo else -u_expr
            condiciones.append((u_expr, es_positivo))
            positivo_por_u.append((u_expr, es_positivo))

        for sg in sign_atoms:
            for u_expr, es_pos in positivo_por_u:
                if sg.args[0] == u_expr:
                    reemplazos_rama[sg] = sp.Integer(1) if es_pos else sp.Integer(-1)
                    break

        f_rama = f_expr.xreplace(reemplazos_rama)
        try:
            f_rama_prima = sp.diff(f_rama, x)
            soluciones_rama = sp.solve(sp.Eq(f_rama_prima, 0), x)
        except Exception:
            soluciones_rama = []

        for s in soluciones_rama:
            try:
                if sp.im(sp.N(s)) != 0:
                    continue
                s_real = sp.re(s)
            except Exception:
                continue
            if dominio is not None and dominio.contains(s_real) == False:
                continue
            if all(_condicion_de_rama_valida(u_e, s_real, pos) for u_e, pos in condiciones):
                if s_real not in vistos:
                    vistos.append(s_real)
                    puntos_totales.append(s_real)

    return puntos_totales


def limpiar_sintaxis_matematica(expresion: str) -> str:
    exp = expresion.strip()
    if "=" in exp:
        parts = exp.split("=")
        exp = parts[-1]

    exp = re.sub(r'^[a-zA-Z_][a-zA-Z0-9_]*\s*\([xX]\)\s*=', '', exp)

    patron_super = '[' + ''.join(SUPERINDICES_A_NORMAL.keys()) + ']+'

    def _convertir_superindice(m):
        digitos = ''.join(SUPERINDICES_A_NORMAL.get(ch, '') for ch in m.group(0))
        return f'**{digitos}'

    exp = re.sub(patron_super, _convertir_superindice, exp)

    exp = re.sub(r'\be\s*\^\s*(\((?:[^()]|\([^()]*\))*\)|[a-zA-Z0-9_\-]+)', r'exp(\1)', exp, flags=re.IGNORECASE)
    exp = exp.replace("^", "**")

    placeholders = {}
    for i, fn in enumerate(FUNCIONES_CONOCIDAS):
        token = chr(0xE000 + i)
        patron = re.compile(r'\b' + fn + r'(?=\s*\()')
        if patron.search(exp):
            exp = patron.sub(token, exp)
            placeholders[token] = fn

    exp = re.sub(r'([a-zA-Z0-9\)])(\(|[\uE000-\uE0FF])', r'\1*\2', exp)
    exp = re.sub(r'(\))([a-zA-Z0-9])', r'\1*\2', exp)
    exp = re.sub(r'(\d)([a-zA-Z])', r'\1*\2', exp)
    exp = re.sub(r'([a-zA-Z])(\d+)', r'\1**\2', exp)

    for token, fn in placeholders.items():
        exp = exp.replace(token, fn)

    return exp.strip()


SUPERINDICES = {'0': '⁰', '1': '¹', '2': '²', '3': '³', '4': '⁴',
                '5': '⁵', '6': '⁶', '7': '⁷', '8': '⁸', '9': '⁹', '-': '⁻'}


def formatear_titulo_bonito(f_expr) -> str:
    """
    Texto del botón de historial a partir de la expresión interpretada
    (texto plano: los botones de Streamlit no aceptan LaTeX).
    """
    limpio = latex_sin_sign(f_expr)

    def sup_repl(m):
        return ''.join(SUPERINDICES.get(ch, ch) for ch in m.group(1))
    limpio = re.sub(r'\^\{(-?\d+)\}', sup_repl, limpio)
    limpio = re.sub(r'\^(-?\d)', sup_repl, limpio)

    limpio = re.sub(r'\\frac\{([^{}]*)\}\{([^{}]*)\}', r'(\1)/(\2)', limpio)
    limpio = re.sub(r'\^\{([^{}]+)\}', r'^(\1)', limpio)

    limpio = re.sub(r'\\sqrt\{([^{}]+)\}', r'√(\1)', limpio)
    limpio = re.sub(r'\\log\b', 'ln', limpio)
    limpio = re.sub(r'\\ln\b', 'ln', limpio)
    limpio = re.sub(r'\\exp\b', 'exp', limpio)
    limpio = re.sub(r'\\left|\\right', '', limpio)
    limpio = re.sub(r'\\,|\\;|\\quad|\\!', '', limpio)
    limpio = limpio.replace(r'\cdot', '·')
    limpio = limpio.replace(r'\times', '×')
    limpio = re.sub(r'\\text\{([^{}]*)\}', r'\1', limpio)
    limpio = limpio.replace(r'\operatorname{sign}', '[±1 por tramo]')
    limpio = limpio.replace(r'\mathrm{', '').replace(r'\,', ' ')
    limpio = limpio.replace("{", "").replace("}", "")

    limpio = re.sub(r'(?<=[0-9)])\s+(?=[a-zA-Z(])', '', limpio)
    limpio = re.sub(r'^-\s+', '-', limpio)
    limpio = re.sub(r'\(-\s+', '(-', limpio)
    limpio = re.sub(r'\(\s+', '(', limpio)
    limpio = re.sub(r'\s+\)', ')', limpio)
    limpio = re.sub(r'\s+', ' ', limpio).strip()

    if len(limpio) > 30:
        limpio = limpio[:28].rstrip() + "..."

    return f"f(x): {limpio}"


def formatear_numero(valor, expr_sympy=None):
    """Formatea un número: exacto si es racional/entero, 4 decimales si no."""
    try:
        if expr_sympy is not None and (expr_sympy.is_Rational or expr_sympy.is_Integer):
            return f"{float(valor):g}"
    except Exception:
        pass
    try:
        return f"{float(valor):g}" if float(valor).is_integer() else f"{float(valor):.4f}"
    except Exception:
        return str(valor)


# ------------------------------------------------------------------
# UTILIDADES DE VALORES Y FORMATO
# ------------------------------------------------------------------
def valor_f_exacto(f_expr, xv):
    """f(xv) como expresión exacta de sympy, o None si no es real y finita."""
    try:
        v = sp.simplify(f_expr.subs(x, xv))
        vn = sp.N(v)
        if vn.is_real is True and vn.is_finite is True:
            return v
    except Exception:
        pass
    return None


def _a_float(v):
    return float(sp.N(v))


def texto_valor(v):
    """Texto plano limpio de un valor: entero, fracción p/q, o forma simbólica corta."""
    try:
        if v is None:
            return "—"
        if v == sp.oo:
            return "+∞"
        if v == -sp.oo:
            return "−∞"
        if v.is_Integer:
            return str(int(v))
        if v.is_Rational:
            return f"{int(v.p)}/{int(v.q)}"
        s = str(v).replace("sqrt", "√").replace("**", "^").replace("*", "·")
        s = re.sub(r'exp\((.*)\)', r'e^(\1)', s)
        return s
    except Exception:
        return str(v)


def latex_valor(v):
    """LaTeX de un valor exacto; si es irracional añade su aproximación decimal."""
    try:
        s = latex_sin_sign(v)
        if v.is_Rational:
            return s
        return f"{s} \\approx {_a_float(v):.4f}"
    except Exception:
        return str(v)


def latex_limite(lim):
    try:
        if lim == sp.oo:
            return "+\\infty"
        if lim == -sp.oo:
            return "-\\infty"
        return latex_sin_sign(lim)
    except Exception:
        return str(lim)


def calcular_limite(f_expr, punto, direccion=None):
    """Límite con sympy y, si no concluye, estimación numérica robusta."""
    try:
        if direccion is None:
            lim = sp.limit(f_expr, x, punto)
        else:
            lim = sp.limit(f_expr, x, punto, dir=direccion)
        if lim in (sp.oo, -sp.oo) or lim.is_finite is True:
            return lim
    except Exception:
        pass

    try:
        if punto in (sp.oo, -sp.oo):
            sg = 1 if punto == sp.oo else -1
            p1, p2 = sg * 10 ** 6, sg * 10 ** 8
        else:
            sg = 1 if direccion == '+' else -1
            p1 = punto + sg * sp.Rational(1, 10 ** 6)
            p2 = punto + sg * sp.Rational(1, 10 ** 8)
        v1 = float(sp.N(f_expr.subs(x, p1), 30))
        v2 = float(sp.N(f_expr.subs(x, p2), 30))
        if abs(v2) > 1e5 and abs(v2) > 1.5 * abs(v1):
            return sp.oo if v2 > 0 else -sp.oo
        if abs(v2 - v1) < 1e-3 * (1 + abs(v2)):
            return sp.Rational(str(round(v2, 6))).limit_denominator(1000)
    except Exception:
        pass
    return None


# ------------------------------------------------------------------
# DOMINIO
# ------------------------------------------------------------------
def calcular_dominio(f_expr, intervalo_usuario=None):
    """Dominio real de f (con la restricción del intervalo del usuario si existe)."""
    try:
        dominio = continuous_domain(f_expr, x, sp.S.Reals)
    except Exception:
        dominio = sp.S.Reals
        try:
            denom_expr = sp.denom(sp.together(f_expr))
            if denom_expr != 1:
                ceros = [c for c in sp.solve(sp.Eq(denom_expr, 0), x) if sp.im(sp.N(c)) == 0]
                if ceros:
                    dominio = sp.S.Reals - sp.FiniteSet(*ceros)
        except Exception:
            dominio = sp.S.Reals
    if intervalo_usuario is not None:
        a, b, lo, ro = desempacar_intervalo(intervalo_usuario)
        try:
            dominio = dominio.intersect(sp.Interval(a, b, lo, ro))
        except Exception:
            pass
    return dominio


def describir_restricciones(f_expr):
    restricciones = []

    denom_expr = sp.denom(sp.together(f_expr))
    if denom_expr != 1:
        try:
            ceros = sp.solve(sp.Eq(denom_expr, 0), x)
            ceros_reales = [c for c in ceros if sp.im(sp.N(c)) == 0]
        except Exception:
            ceros_reales = []
        if ceros_reales:
            vals = ", ".join(f"x \\neq {latex_sin_sign(c)}" for c in ceros_reales)
            restricciones.append(("El denominador no puede anularse:", vals))

    for p in f_expr.atoms(sp.Pow):
        try:
            if p.exp.is_Rational and p.exp.q == 2:
                if p.exp < 0:
                    restricciones.append((
                        "El radicando debe ser mayor que cero:",
                        f"{latex_sin_sign(p.base)} > 0"
                    ))
                else:
                    restricciones.append((
                        "El radicando debe ser mayor o igual a cero:",
                        f"{latex_sin_sign(p.base)} \\geq 0"
                    ))
        except Exception:
            pass

    for lg in f_expr.atoms(sp.log):
        restricciones.append((
            "El argumento del logaritmo debe ser mayor que cero:",
            f"{latex_sin_sign(lg.args[0])} > 0"
        ))

    return restricciones


def extremos_finitos_de_dominio(dominio):
    """Lista de tuplas (valor, es_abierto) para los extremos finitos del dominio."""
    extremos = []
    if dominio is None:
        return extremos
    if isinstance(dominio, sp.Interval):
        if dominio.start.is_finite:
            extremos.append((dominio.start, bool(dominio.left_open)))
        if dominio.end.is_finite:
            extremos.append((dominio.end, bool(dominio.right_open)))
    elif isinstance(dominio, (sp.Union, sp.FiniteSet)):
        for arg in dominio.args:
            extremos.extend(extremos_finitos_de_dominio(arg))
    vistos = []
    unicos = []
    for val, abierto in extremos:
        if val not in vistos:
            vistos.append(val)
            unicos.append((val, abierto))
    return unicos


def obtener_intervalos_dominio(dominio):
    """Descompone el dominio en una lista de sp.Interval (aplana Uniones)."""
    if dominio is None:
        return []
    if isinstance(dominio, sp.Interval):
        return [dominio]
    if isinstance(dominio, sp.Union):
        intervalos = []
        for arg in dominio.args:
            intervalos.extend(obtener_intervalos_dominio(arg))
        intervalos.sort(key=lambda I: float(sp.N(I.start)) if I.start.is_finite else float('-inf'))
        return intervalos
    return []


def es_borde_del_dominio(dominio, p):
    for intervalo in obtener_intervalos_dominio(dominio):
        if intervalo.start == p or intervalo.end == p:
            return True
    return False


def analizar_fronteras_dominio(f_expr, dominio):
    """
    Analiza cada frontera de cada intervalo del dominio:
    - 'cerrado'  : el borde pertenece al dominio.
    - 'abierto'  : borde finito que no pertenece (límite lateral).
    - 'infinito' : frontera en +-oo (límite al infinito).
    """
    fronteras = []
    for intervalo in obtener_intervalos_dominio(dominio):
        if intervalo.start == intervalo.end:
            continue
        candidatos = [
            (intervalo.start, bool(intervalo.left_open), '+'),
            (intervalo.end, bool(intervalo.right_open), '-'),
        ]
        for val, abierto, direccion in candidatos:
            if val in (sp.oo, -sp.oo):
                lim = calcular_limite(f_expr, val)
                fronteras.append({'valor': val, 'tipo': 'infinito', 'limite': lim, 'dir': None})
            elif abierto:
                lim = calcular_limite(f_expr, val, direccion)
                fronteras.append({'valor': val, 'tipo': 'abierto', 'limite': lim, 'dir': direccion})
            else:
                fronteras.append({'valor': val, 'tipo': 'cerrado', 'limite': None, 'dir': None})

    vistos = set()
    unicas = []
    for f in fronteras:
        clave = (str(f['valor']), f['tipo'], f['dir'])
        if clave not in vistos:
            vistos.add(clave)
            unicas.append(f)
    return unicas


# ------------------------------------------------------------------
# TABLA DE SIGNOS / MONOTONÍA
# ------------------------------------------------------------------
def _punto_de_prueba(lo, hi):
    if lo == -sp.oo and hi == sp.oo:
        return sp.Integer(0)
    if lo == -sp.oo:
        return hi - 1
    if hi == sp.oo:
        return lo + 1
    return (lo + hi) / 2


def _signo_numerico(f_prime, punto):
    try:
        v = sp.N(f_prime.subs(x, punto))
        if v.is_real is not True:
            return None
        vf = float(v)
        if abs(vf) < 1e-12:
            return 0
        return 1 if vf > 0 else -1
    except Exception:
        return None


def construir_tramos(f_prime, dominio, cortes, signo_fn=None):
    """
    Parte cada intervalo del dominio en los puntos de corte y evalúa el
    signo en un punto de prueba de cada subintervalo. Si se da signo_fn
    (función de un punto), se usa en lugar de evaluar f_prime.
    """
    tramos = []
    for idx, intervalo in enumerate(obtener_intervalos_dominio(dominio)):
        a, b = intervalo.start, intervalo.end
        if a == b:
            continue
        internos = [c for c in cortes if bool(c > a) and bool(c < b)]
        internos.sort(key=lambda c: float(sp.N(c)))
        puntos = [a] + internos + [b]
        for lo, hi in zip(puntos[:-1], puntos[1:]):
            t = _punto_de_prueba(lo, hi)
            tramos.append({
                'lo': lo,
                'hi': hi,
                'izq_cerrado': bool(lo == a and not intervalo.left_open),
                'der_cerrado': bool(hi == b and not intervalo.right_open),
                'signo': (signo_fn(t) if signo_fn else _signo_numerico(f_prime, t)),
                'idx': idx,
            })
    return tramos


# ------------------------------------------------------------------
# SEGUNDA DERIVADA (funciona también con valor absoluto)
# ------------------------------------------------------------------
def ramas_abs_expr(expr):
    """[(condiciones[(u, positivo)], expr_sin_abs)]; reemplaza sign(u) por ±1."""
    abs_atoms = sorted(expr.atoms(sp.Abs), key=sp.default_sort_key)
    if not abs_atoms:
        return [([], expr)]
    ramas = []
    for m in range(2 ** len(abs_atoms)):
        rep, conds = {}, []
        for i, ab in enumerate(abs_atoms):
            u = ab.args[0]
            pos = bool((m >> i) & 1)
            rep[ab] = u if pos else -u
            conds.append((u, pos))
            for sg in expr.atoms(sp.sign):
                if sg.args[0] == u:
                    rep[sg] = sp.Integer(1) if pos else sp.Integer(-1)
        ramas.append((conds, expr.xreplace(rep)))
    return ramas


def derivada2_en_punto(f_expr, p):
    """f''(p) exacta, usando la rama de |·| a la que pertenece p."""
    for conds, e in ramas_abs_expr(f_expr):
        if all(_condicion_de_rama_valida(u, p, pos) for u, pos in conds):
            try:
                v = sp.simplify(sp.diff(e, x, 2).subs(x, p))
                if sp.N(v).is_real:
                    return v
            except Exception:
                pass
    return None


def _signo_f2(f_expr):
    def fn(t):
        v = derivada2_en_punto(f_expr, t)
        if v is None:
            return None
        try:
            vf = float(sp.N(v))
        except Exception:
            return None
        return 0 if abs(vf) < 1e-12 else (1 if vf > 0 else -1)
    return fn


def ceros_segunda_derivada(f_expr, dominio):
    """Puntos donde f''=0 o f'' no existe (candidatos a inflexión)."""
    pts = []
    for conds, e in ramas_abs_expr(f_expr):
        try:
            d2 = sp.together(sp.diff(e, x, 2))
            cands = sp.solve(sp.Eq(sp.numer(d2), 0), x) + sp.solve(sp.Eq(sp.denom(d2), 0), x)
        except Exception:
            continue
        for s in cands:
            try:
                if sp.im(sp.N(s)) != 0:
                    continue
                s = sp.re(s)
                if dominio.contains(s) == False:
                    continue
                if all(_condicion_de_rama_valida(u, s, pos) for u, pos in conds) \
                        and not any(s == q for q in pts):
                    pts.append(s)
            except Exception:
                pass
    return pts


# ------------------------------------------------------------------
# ASÍNTOTAS Y GRÁFICA
# ------------------------------------------------------------------
def calcular_asintotas(f_expr, fronteras):
    vert, hor, obl = [], [], []
    for fr in fronteras:
        lim = fr.get('limite')
        if fr['tipo'] == 'abierto' and lim in (sp.oo, -sp.oo) and fr['valor'] not in vert:
            vert.append(fr['valor'])
        elif fr['tipo'] == 'infinito' and lim is not None:
            if lim.is_finite is True:
                hor.append((fr['valor'], lim))
            else:
                try:
                    m = sp.limit(f_expr / x, x, fr['valor'])
                    if m.is_finite and m != 0:
                        b = sp.limit(f_expr - m * x, x, fr['valor'])
                        if b.is_finite:
                            obl.append((fr['valor'], m, b))
                except Exception:
                    pass
    return vert, hor, obl


def graficar_resumen(res):
    import numpy as np
    import matplotlib.pyplot as plt
    f = sp.lambdify(x, res['f_orig'], 'numpy')
    puntos = [float(sp.N(c['x'])) for c in res.get('candidatos', [])]
    lo = (min(puntos) if puntos else -3) - 2
    hi = (max(puntos) if puntos else 3) + 2
    xs = np.linspace(lo, hi, 2500)
    with np.errstate(all='ignore'):
        ys = np.broadcast_to(np.array(f(xs), dtype=complex), xs.shape)
    ys = np.where(np.abs(ys.imag) < 1e-9, ys.real, np.nan)
    if np.isfinite(ys).any():
        a, b = np.nanpercentile(ys, [2, 98])
        pad = max(1.0, (b - a) * 0.3)
        ys = np.where(np.abs(ys) > 10 * max(abs(a), abs(b), 1), np.nan, ys)
        ylim = (a - pad, b + pad)
    else:
        ylim = (-5, 5)
    fig, ax = plt.subplots(figsize=(7, 4))
    ax.plot(xs, ys, color="#007A33", lw=2)
    ax.axhline(0, color="#94A3B8", lw=0.8)
    ax.axvline(0, color="#94A3B8", lw=0.8)
    for xv, yv, tipo in res.get('extremos_locales', []):
        ax.plot(float(sp.N(xv)), float(sp.N(yv)), "o", ms=8,
                color="#16A34A" if "máx" in tipo else "#DC2626", label=tipo)
    for xv, yv in res.get('inflexion', []):
        ax.plot(float(sp.N(xv)), float(sp.N(yv)), "s", ms=7, color="#2563EB", label="inflexión")
    vert, hor, _ = res.get('asintotas', ([], [], []))
    for v in vert:
        ax.axvline(float(sp.N(v)), ls="--", color="#D97706", lw=1)
    for _, l in hor:
        ax.axhline(float(sp.N(l)), ls="--", color="#D97706", lw=1)
    ax.set_ylim(*ylim)
    ax.set_xlim(lo, hi)
    ax.grid(alpha=0.25)
    h, l = ax.get_legend_handles_labels()
    if h:
        ax.legend(dict(zip(l, h)).values(), dict(zip(l, h)).keys(), fontsize=8)
    return fig


def _fmt_intervalo(lo, hi, cerr_izq=False, cerr_der=False):
    a = "−∞" if lo == -sp.oo else texto_valor(lo)
    b = "∞" if hi == sp.oo else texto_valor(hi)
    ap = "[" if (cerr_izq and lo != -sp.oo) else "("
    cp = "]" if (cerr_der and hi != sp.oo) else ")"
    return f"{ap}{a}, {b}{cp}"


def _fmt_dominio_tabla(dominio):
    if dominio is None:
        return "ℝ"
    if dominio == sp.S.Reals:
        return "ℝ"
    intervalos = obtener_intervalos_dominio(dominio)
    if not intervalos:
        return str(dominio)
    return " ∪ ".join(
        _fmt_intervalo(I.start, I.end, not I.left_open, not I.right_open) for I in intervalos
    )


def _fmt_lista_puntos_tabla(puntos):
    if not puntos:
        return "Ninguno"
    return ", ".join(f"x = {texto_valor(p)}" for p in puntos)


def _fmt_limite_tabla(lim):
    if lim is None:
        return "—"
    return texto_valor(lim)


def generar_tabla_resumen(dominio, puntos_criticos, puntos_singulares,
                           limites, extremos_locales,
                           max_c, min_c, max_existe, min_existe, inflexion=None):
    """Tabla resumen final en Markdown."""
    filas = []
    filas.append("| Concepto | Resultado analítico |")
    filas.append("|---|---|")
    filas.append(f"| Dominio | Dom(f) = {_fmt_dominio_tabla(dominio)} |")
    filas.append(f"| Puntos estacionarios | {_fmt_lista_puntos_tabla(puntos_criticos)} |")
    filas.append(f"| Puntos singulares | {_fmt_lista_puntos_tabla(puntos_singulares)} |")

    if limites:
        lim_str = "; ".join(f"{etq}: {_fmt_limite_tabla(lim)}" for etq, lim in limites)
    else:
        lim_str = "Sin bordes abiertos ni infinitos"
    filas.append(f"| Límites | {lim_str} |")

    if extremos_locales:
        loc_str = "; ".join(
            f"{tipo} en x = {texto_valor(xv)} (f = {texto_valor(yv)})"
            for xv, yv, tipo in extremos_locales
        )
    else:
        loc_str = "Ninguno"
    filas.append(f"| Extremos locales | {loc_str} |")
    if inflexion:
        infl_str = "; ".join(f"x = {texto_valor(xv)} (f = {texto_valor(yv)})" for xv, yv in inflexion)
    else:
        infl_str = "Ninguno"
    filas.append(f"| Puntos de inflexión | {infl_str} |")

    if max_existe and max_c is not None:
        max_str = f"f({texto_valor(max_c['x'])}) = {texto_valor(max_c['y'])}"
    else:
        max_str = "No existe"
    filas.append(f"| Máximo absoluto | {max_str} |")

    if min_existe and min_c is not None:
        min_str = f"f({texto_valor(min_c['x'])}) = {texto_valor(min_c['y'])}"
    else:
        min_str = "No existe"
    filas.append(f"| Mínimo absoluto | {min_str} |")

    return "\n".join(filas)


# ------------------------------------------------------------------
# GENERACIÓN DE TIPS
# ------------------------------------------------------------------
def generar_tips(f_expr_visual, intervalo_usuario=None):
    """Lista de tuplas (titulo_paso, contenido), una por cada uno de los 6 pasos."""
    f_expr = hacer_reales_potencias_fraccionarias(f_expr_visual)
    tips = []

    denom_expr = sp.denom(sp.together(f_expr_visual))
    tiene_denominador = denom_expr != 1

    tiene_raiz_par = False
    tiene_exponente_fraccionario = False
    for p in f_expr_visual.atoms(sp.Pow):
        try:
            if p.exp.is_Rational and p.exp.q == 2:
                tiene_raiz_par = True
            if p.exp.is_Rational and 0 < p.exp < 1:
                tiene_exponente_fraccionario = True
        except Exception:
            pass

    tiene_log = len(f_expr_visual.atoms(sp.log)) > 0
    tiene_abs = len(f_expr_visual.atoms(sp.Abs)) > 0
    dominio = calcular_dominio(f_expr, intervalo_usuario)
    dominio_acotado = bool(extremos_finitos_de_dominio(dominio)) if dominio is not None else False

    partes_dom = []
    if tiene_denominador:
        partes_dom.append("denominadores ≠ 0")
    if tiene_raiz_par:
        partes_dom.append("radicandos ≥ 0")
    if tiene_log:
        partes_dom.append("argumentos de log &gt; 0")
    if partes_dom:
        cont = "Revisa: " + ", ".join(partes_dom) + "."
    else:
        cont = "f(x) es polinómica (o equivalente): su dominio natural es ℝ."
    if intervalo_usuario is not None:
        cont += f" Además, aquí se restringe a x ∈ {texto_intervalo_usuario(intervalo_usuario)}."
    cont += " Antes de derivar, simplifica: factoriza (diferencia de cuadrados o de cubos, factor común) y cancela."
    tips.append(("Paso 1 — Dominio y simplificación", cont))

    tips.append((
        "Paso 2 — Primera derivada",
        "Deriva la función ya simplificada con las reglas de Cálculo 1 (potencias, producto, cociente y cadena) "
        "y factoriza el resultado para ver con claridad dónde se anula."
    ))

    cont_cand = "Estacionarios: resuelve f'(x) = 0."
    if tiene_denominador or tiene_abs or tiene_exponente_fraccionario:
        pistas = []
        if tiene_denominador:
            pistas.append("dónde el denominador de f'(x) se anula")
        if tiene_abs:
            pistas.append("dónde el argumento del valor absoluto es 0")
        if tiene_exponente_fraccionario:
            pistas.append("dónde la base de un exponente fraccionario es 0")
        cont_cand += " Singulares (fáciles de olvidar): busca " + "; ".join(pistas) + " — ahí f'(x) puede no existir aunque f(x) sí."
    else:
        cont_cand += " Aquí no hay puntos donde f'(x) deje de existir dentro del dominio."
    tips.append(("Paso 3 — Puntos críticos", cont_cand))

    if dominio_acotado:
        cont_eval = ("Calcula f(x) en cada punto crítico y también en los bordes cerrados del dominio: "
                     "son candidatos directos.")
    else:
        cont_eval = "Calcula f(x) en cada punto crítico, siempre en la función original."
    tips.append(("Paso 4 — Evaluación", cont_eval))

    tips.append((
        "Paso 5 — Monotonía",
        "Haz la tabla de signos de f'(x): en cada intervalo, positivo = creciente y negativo = decreciente. "
        "El cambio de signo de f' clasifica los extremos locales."
    ))

    tips.append((
        "Paso 6 — Segunda derivada",
        "Deriva f'(x) otra vez. Si f''(x) > 0 la gráfica es cóncava hacia arriba (∪); si f''(x) < 0, hacia abajo (∩). "
        "Donde f'' cambia de signo hay un punto de inflexión, y en un punto estacionario c: f''(c) > 0 es mínimo y f''(c) < 0 es máximo."
    ))

    tips.append((
        "Paso 7 — Límites y asíntotas",
        "Calcula los límites en los bordes abiertos o en ±∞ (no evalúes f(x) en ellos) y deduce las asíntotas verticales, horizontales u oblicuas."
    ))

    tips.append((
        "Paso 8 — Veredicto",
        "El mayor valor alcanzado es el máximo absoluto solo si ningún límite lo supera; igual con el mínimo. "
        "Resume máximos, mínimos e inflexiones sin contradicciones."
    ))

    return tips


# ------------------------------------------------------------------
# ESTADO DE SESIÓN
# ------------------------------------------------------------------
if "historial_problemas" not in st.session_state:
    st.session_state.historial_problemas = []
if "problema_activo" not in st.session_state:
    st.session_state.problema_activo = None
if "mostrar_solucion" not in st.session_state:
    st.session_state.mostrar_solucion = False


def obtener_titulo_historial_exacto(item):
    """Reconstruye el título del historial desde la función original."""
    if item.get('tipo') == 'aplicado':
        return item.get('titulo_bonito', '📐 Problema aplicado')
    original = item.get('funcion_original', '')
    if original:
        try:
            intervalo = extraer_intervalo_usuario(original)
            if intervalo is not None:
                _, _, texto_funcion = intervalo
            else:
                texto_funcion = original
            saneada = limpiar_sintaxis_matematica(texto_funcion)
            expr_visual = sp.sympify(saneada, locals={'x': x, 'ln': sp.log})
            return formatear_titulo_bonito(expr_visual)
        except Exception:
            pass

    return item.get('titulo_bonito', '')


# --- BARRA LATERAL ---
with st.sidebar:
    if os.path.exists("logo_uis.webp"):
        st.image("logo_uis.webp", use_container_width=True)
    else:
        st.info("💡 Sube tu 'logo_uis.webp' al directorio.")
        
    st.markdown("### Sesiones de Chat")
    if st.button("➕ Nueva Conversación", use_container_width=True):
        st.session_state.problema_activo = None
        st.session_state.mostrar_solucion = False
        st.rerun()
        
    st.markdown("---")
    st.markdown("<p style='font-size: 12px; color: #64748B; font-weight: 600;'>HISTORIAL</p>", unsafe_allow_html=True)
    
    if st.session_state.historial_problemas:
        for idx, item in enumerate(reversed(st.session_state.historial_problemas)):
            titulo_historial = obtener_titulo_historial_exacto(item)
            if st.button(titulo_historial, key=f"hist_{idx}", use_container_width=True):
                st.session_state.problema_activo = item
                st.session_state.mostrar_solucion = False
                st.rerun()
    else:
        st.markdown("<p style='font-size: 13px; color: #94A3B8;'>Sin chats previos.</p>", unsafe_allow_html=True)
        
    st.markdown("---")
    st.caption("Universidad Industrial de Santander\nSede Barrancabermeja")

# --- ENCABEZADO ---
col_head1, col_head2 = st.columns([0.12, 0.88])
with col_head1:
    logo_circular = "logo-universidad-industrial-de-santander.webp"
    if os.path.exists(logo_circular):
        st.markdown('<div class="logo-circular">', unsafe_allow_html=True)
        st.image(logo_circular)
        st.markdown('</div>', unsafe_allow_html=True)
    else:
        st.warning("⚠️ Falta 'logo-universidad-industrial-de-santander.webp'")

with col_head2:
    st.markdown("""
        <div class="uis-header" style="margin-bottom: 0px;">
            <div>
                <h3 style="margin: 0; color: white; font-size: 18px;">🧠 Asistente IA de Optimización en una sola variable</h3>
                <span style="font-size: 12px; color: #E2F6EC;">Ingeniería en Inteligencia Artificial • UIS</span>
            </div>
            <span style="background-color: #005E27; padding: 4px 10px; border-radius: 12px; font-size: 11px; font-weight: 600; color: white;">En línea</span>
        </div>
    """, unsafe_allow_html=True)

st.markdown("<br>", unsafe_allow_html=True)


# ------------------------------------------------------------------
# SIMPLIFICACIÓN PREVIA PARA EL ESTUDIANTE
# ------------------------------------------------------------------
def preparar_expresion_estudiante(f_expr_visual):
    """
    Devuelve (expresión_simplificada, pasos_de_simplificación).
    1. Caso clásico (raíz_n(x) - a)/(x - a^n): cambio u = raíz_n(x).
    2. Caso general: factorizar y cancelar factores comunes.
    """
    expr = f_expr_visual
    pasos = []

    try:
        num, den = sp.fraction(sp.together(expr))
        if den != 1 and den.has(x):
            raices = [
                p for p in num.atoms(sp.Pow)
                if p.base == x and p.exp.is_Rational and p.exp.p == 1 and p.exp.q > 1
            ]
            if len(raices) == 1:
                r = raices[0]
                n = int(r.exp.q)
                a = sp.simplify(r - num)
                if (not a.has(x)) and a.is_Rational and sp.simplify(den - (x - a ** n)) == 0:
                    u = sp.Symbol('u', real=True)
                    factor = sum(u ** k * a ** (n - 1 - k) for k in range(n))
                    factor_l = latex_sin_sign(factor)
                    pasos.extend([
                        {"texto": "Antes de derivar, simplificamos con un cambio de variable. "
                                  f"Sea $u = {latex_sin_sign(r)}$, de modo que $x = u^{{{n}}}$ (con $u \\geq 0$)."},
                        {"latex": (
                            f"x - {latex_sin_sign(a ** n)} = u^{{{n}}} - {latex_sin_sign(a ** n)} = "
                            f"\\left({latex_sin_sign(u - a)}\\right)\\left({factor_l}\\right)"
                        )},
                        {"latex": (
                            f"f(x) = \\frac{{{latex_sin_sign(u - a)}}}"
                            f"{{\\left({latex_sin_sign(u - a)}\\right)\\left({factor_l}\\right)}}"
                            f" = \\frac{{1}}{{{factor_l}}}"
                        )},
                        {"texto": "Regresamos a $x$ (sustituyendo $u$ por la raíz) y seguimos con la expresión simplificada, "
                                  "válida en el dominio de la función original:"},
                        {"latex": f"f(x) = \\frac{{1}}{{{latex_sin_sign(factor.subs(u, r))}}}"},
                    ])
                    return sp.Pow(factor.subs(u, r), -1), pasos
    except Exception:
        pass

    try:
        num, den = sp.fraction(sp.together(expr))
        if den != 1 and den.has(x):
            cancelada = sp.cancel(sp.together(expr))
            num_c, den_c = sp.fraction(cancelada)
            hubo_cancelacion = sp.simplify(den / den_c).has(x)
            if hubo_cancelacion:
                fn = sp.factor(num)
                fd = sp.factor(den)
                pasos.extend([
                    {"texto": "Antes de derivar, factorizamos numerador y denominador y cancelamos los factores comunes:"},
                    {"latex": f"f(x) = \\frac{{{latex_sin_sign(fn)}}}{{{latex_sin_sign(fd)}}}"},
                    {"latex": f"f(x) = {latex_sin_sign(cancelada)}"},
                    {"texto": "La expresión simplificada es equivalente a $f(x)$ solo en el dominio de la función original; "
                              "los valores excluidos siguen excluidos."},
                ])
                return cancelada, pasos
    except Exception:
        pass

    return expr, pasos


# ------------------------------------------------------------------
# TRAMOS POR VALOR ABSOLUTO (SOLO SI LA FUNCIÓN ORIGINAL LO TRAE)
# ------------------------------------------------------------------
def ramas_valor_absoluto(expr):
    """Lista de (condición_latex, expresión_sin_valor_absoluto) o [(None, expr)]."""
    abs_atoms = sorted(expr.atoms(sp.Abs), key=sp.default_sort_key)
    if not abs_atoms:
        return [(None, expr)]
    ramas = []
    geq = "\\geq"
    for mascara in range(2 ** len(abs_atoms)):
        reemplazos = {}
        conds = []
        for i, ab in enumerate(abs_atoms):
            u = ab.args[0]
            positivo = bool((mascara >> i) & 1)
            reemplazos[ab] = u if positivo else -u
            conds.append(f"{latex_sin_sign(u)} {geq if positivo else '<'} 0")
        ramas.append((",\\; ".join(conds), expr.xreplace(reemplazos)))
    return ramas


def latex_por_tramos(nombre, ramas):
    filas = [
        f"{latex_sin_sign(e)} & \\text{{si }} {cond}"
        for cond, e in ramas
    ]
    return nombre + " = \\begin{cases} " + " \\\\ ".join(filas) + " \\end{cases}"


def describir_regla_derivacion(expr):
    """Frase corta que indica qué regla de Cálculo 1 se usa."""
    try:
        num, den = sp.fraction(sp.together(expr))
        if den.has(x) and num.has(x):
            return "Como $f(x)$ es un cociente, usamos la regla del cociente y luego factorizamos."
        if den.has(x):
            return "Escribimos $f(x)$ como una potencia negativa y aplicamos la regla de la cadena; luego factorizamos."
        if expr.is_Mul and sum(1 for a in expr.args if a.has(x)) >= 2:
            return "Como $f(x)$ es un producto, usamos la regla del producto y luego factorizamos."
        if expr.is_Add:
            return "Derivamos término a término con la regla de las potencias y factorizamos el resultado."
    except Exception:
        pass
    return "Aplicamos las reglas de potencias y de la cadena, y factorizamos el resultado."


def razon_punto_singular(derivadas_est, f_est, p):
    """Texto que explica por qué f'(x) no existe en el punto singular p."""
    try:
        for _, _, d in derivadas_est:
            den = sp.denom(sp.together(d))
            if den != 1 and sp.simplify(den.subs(x, p)) == 0:
                return "el denominador de $f'(x)$ se anula en ese punto"
    except Exception:
        pass
    try:
        for ab in f_est.atoms(sp.Abs):
            if sp.simplify(ab.args[0].subs(x, p)) == 0:
                return "el valor absoluto cambia de tramo en ese punto (esquina)"
    except Exception:
        pass
    return "$f'(x)$ no está definida en ese punto"


# ------------------------------------------------------------------
# CONSTRUCCIÓN DE LA SOLUCIÓN NARRATIVA (6 PASOS)
# ------------------------------------------------------------------
def construir_pasos_narrativos(f_expr_calculo, intervalo_usuario=None, f_expr_visual=None, resumen_out=None):
    pasos = []

    # f_orig: función con corrección real de potencias fraccionarias (motor interno).
    # f_vis : la función tal como la ve el estudiante.
    # f_est : f_vis simplificada algebraicamente (se deriva esta).
    # f_calc: f_est con corrección real, para clasificar y evaluar signos.
    f_orig = f_expr_calculo
    f_vis = f_expr_visual if f_expr_visual is not None else f_expr_calculo
    f_est, pasos_simplificacion = preparar_expresion_estudiante(f_vis)
    f_calc = hacer_reales_potencias_fraccionarias(f_est)
    dominio = calcular_dominio(f_orig, intervalo_usuario)
    f_prime_calc = sp.diff(f_calc, x)

    ramas_f = ramas_valor_absoluto(f_est)
    tiene_tramos = ramas_f[0][0] is not None
    derivadas_est = []  # (condición, derivada_cruda, derivada_factorizada)
    for cond, expr_r in ramas_f:
        d_raw = sp.diff(expr_r, x)
        try:
            d_fact = sp.factor(sp.together(d_raw))
        except Exception:
            d_fact = d_raw
        derivadas_est.append((cond, d_raw, d_fact))

    # ================================================================
    # PASO 1 — DOMINIO Y SIMPLIFICACIÓN ALGEBRAICA
    # ================================================================
    pasos.append({"header": "Paso 1 — Dominio y simplificación algebraica"})

    restricciones = describir_restricciones(f_vis)
    if restricciones:
        pasos.append({"texto": "Condiciones que debe cumplir $x$ para que $f(x)$ exista:"})
        pasos.append({"latex": " \\\\\n".join([lat for _, lat in restricciones])})
    else:
        pasos.append({"texto": "La expresión no tiene denominadores, raíces de índice par ni logaritmos, "
                               "así que no impone restricciones: el dominio es todo $\\mathbb{R}$."})

    if intervalo_usuario is not None:
        pasos.append({"texto": f"Además, el problema restringe $x$ al intervalo {texto_intervalo_usuario(intervalo_usuario)}."})

    if dominio == sp.S.Reals:
        pasos.append({"latex": "\\text{Dom}(f) = \\mathbb{R}"})
    elif dominio == sp.S.EmptySet:
        pasos.append({"texto": "Con estas condiciones no queda ningún valor de $x$ permitido."})
    else:
        pasos.append({"latex": f"\\text{{Dom}}(f) = \\boxed{{{latex_sin_sign(dominio)}}}"})

    if pasos_simplificacion:
        for paso_s in pasos_simplificacion:
            pasos.append(paso_s)
    else:
        pasos.append({"texto": "La función ya está en su forma más simple: no hay factores comunes que cancelar."})

    if tiene_tramos:
        pasos.append({
            "texto": "Como $f(x)$ contiene un valor absoluto, la escribimos por tramos "
                     "(si lo de adentro es positivo se deja igual; si es negativo se cambia de signo):",
            "latex": latex_por_tramos("f(x)", ramas_f),
        })

    # ================================================================
    # PASO 2 — PRIMERA DERIVADA
    # ================================================================
    pasos.append({"header": "Paso 2 — Primera derivada"})

    if not tiene_tramos:
        _, d_raw, d_fact = derivadas_est[0]
        lineas = [f"f'(x) = {latex_sin_sign(d_raw)}"]
        if latex_sin_sign(d_fact) != latex_sin_sign(d_raw):
            lineas.append(f"f'(x) = {latex_sin_sign(d_fact)}")
        pasos.append({
            "texto": describir_regla_derivacion(f_est),
            "latex": " \\\\\n".join(lineas),
        })
    else:
        filas = [
            f"{latex_sin_sign(d_fact)} & \\text{{si }} {cond}"
            for cond, _, d_fact in derivadas_est
        ]
        pasos.append({
            "texto": "Derivamos cada tramo por separado con las reglas de Cálculo 1:",
            "latex": "f'(x) = \\begin{cases} " + " \\\\ ".join(filas) + " \\end{cases}",
        })

    # ================================================================
    # PASO 3 — PUNTOS CRÍTICOS
    # ================================================================
    pasos.append({"header": "Paso 3 — Puntos críticos"})

    puntos_criticos = resolver_puntos_estacionarios(f_calc, dominio)
    puntos_criticos.sort(key=lambda p: float(sp.N(p)))
    puntos_singulares = encontrar_puntos_no_diferenciables(f_calc, f_prime_calc, dominio)
    puntos_singulares = [p for p in puntos_singulares if p not in puntos_criticos]
    puntos_singulares.sort(key=lambda p: float(sp.N(p)))

    pasos.append({"texto": "**Puntos estacionarios** (donde $f'(x) = 0$):"})
    eq_lineas = []
    nota_constante = None
    for cond, _, d_fact in derivadas_est:
        try:
            numerador = sp.numer(sp.together(d_fact))
            denominador = sp.denom(sp.together(d_fact))
        except Exception:
            numerador, denominador = d_fact, sp.Integer(1)
        sufijo = f" \\quad \\left(\\text{{si }} {cond}\\right)" if cond else ""
        if not numerador.has(x):
            nota_constante = numerador
            continue
        if denominador != 1:
            eq_lineas.append(f"{latex_sin_sign(numerador)} = 0{sufijo}")
        else:
            eq_lineas.append(f"{latex_sin_sign(d_fact)} = 0{sufijo}")

    if eq_lineas:
        dens = [sp.denom(sp.together(d)) for _, _, d in derivadas_est]
        if any(dn != 1 and dn.func != sp.exp for dn in dens):
            pasos.append({"texto": "Una fracción vale $0$ solo si su numerador vale $0$:"})
        elif any(dn.func == sp.exp for dn in dens):
            pasos.append({"texto": "El factor exponencial nunca vale $0$, así que basta igualar a $0$ el otro factor:"})
        pasos.append({"latex": " \\\\\n".join(eq_lineas)})
    elif nota_constante is not None:
        pasos.append({"texto": f"El numerador de $f'(x)$ es la constante ${latex_sin_sign(nota_constante)}$, "
                               "que nunca vale $0$."})

    if puntos_criticos:
        pc_strs = [f"x = \\boxed{{{latex_sin_sign(pc)}}}" for pc in puntos_criticos]
        pasos.append({"texto": "Soluciones dentro del dominio: $" + "$, $".join(pc_strs) + "$."})
    else:
        pasos.append({"texto": "La ecuación $f'(x) = 0$ no tiene soluciones reales en el dominio: "
                               "**no hay puntos estacionarios**."})

    pasos.append({"texto": "**Puntos singulares** (donde $f(x)$ existe pero $f'(x)$ no):"})
    if puntos_singulares:
        for p in puntos_singulares:
            vp = valor_f_exacto(f_orig, p)
            extra_val = f" (aquí $f({latex_sin_sign(p)}) = {latex_valor(vp)}$)" if vp is not None else ""
            extra_borde = " Además, es un borde del dominio." if es_borde_del_dominio(dominio, p) else ""
            pasos.append({
                "texto": f"En $x = \\boxed{{{latex_sin_sign(p)}}}$ la función está definida{extra_val}, "
                         f"pero {razon_punto_singular(derivadas_est, f_est, p)}. "
                         f"Por eso es un punto singular.{extra_borde}"
            })
    else:
        pasos.append({"texto": "Todos los puntos del dominio donde $f(x)$ existe también tienen derivada: "
                               "**no hay puntos singulares**."})

    # ================================================================
    # PASO 4 — EVALUACIÓN EN LA FUNCIÓN ORIGINAL
    # ================================================================
    pasos.append({"header": "Paso 4 — Evaluación de los puntos críticos en f(x)"})

    fronteras = analizar_fronteras_dominio(f_calc, dominio)

    candidatos = []  # {'x', 'y', 'tipo'}

    def _agregar_candidato(p, tipo):
        if any(c['x'] == p for c in candidatos):
            return
        v = valor_f_exacto(f_orig, p)
        if v is None:
            return
        candidatos.append({'x': p, 'y': v, 'tipo': tipo})

    for p in puntos_criticos:
        _agregar_candidato(p, 'estacionario')
    for p in puntos_singulares:
        _agregar_candidato(p, 'singular')
    for fr in fronteras:
        if fr['tipo'] == 'cerrado':
            _agregar_candidato(fr['valor'], 'borde')
    candidatos.sort(key=lambda c: float(sp.N(c['x'])))

    etiquetas_tipo = {
        'estacionario': "punto estacionario",
        'singular': "punto singular",
        'borde': "borde del dominio",
    }

    if candidatos:
        intro_eval = "Evaluamos la función original en cada punto crítico y en cada borde que pertenece al dominio:"
        if intervalo_usuario is not None:
            a_i, b_i, lo_i, ro_i = desempacar_intervalo(intervalo_usuario)
            try:
                if (not lo_i) and (not ro_i) and a_i.is_finite and b_i.is_finite and dominio == sp.Interval(a_i, b_i):
                    intro_eval = (f"Como $f$ es continua en el intervalo cerrado $[{texto_valor(a_i)}, {texto_valor(b_i)}]$, "
                                  "el Teorema del Valor Extremo garantiza que existen máximo y mínimo absolutos. "
                                  "Se hallan comparando $f$ en los puntos críticos y en los extremos del intervalo:")
            except Exception:
                pass
        pasos.append({"texto": intro_eval})
        for c in candidatos:
            lc = latex_sin_sign(c['x'])
            lv = latex_valor(c['y'])
            lat = f"f({lc}) = \\boxed{{{lv}}}"
            pasos.append({
                "texto": f"En $x = {lc}$ ({etiquetas_tipo[c['tipo']]}):",
                "latex": lat,
            })
    else:
        pasos.append({"texto": "No hay puntos críticos ni bordes que pertenezcan al dominio, así que no hay valores "
                               "que evaluar. El comportamiento de $f$ se describe con los límites del Paso 7."})

    # ================================================================
    # PASO 5 — MONOTONÍA, CONCAVIDAD, LÍMITES Y ASÍNTOTAS
    # ================================================================
    pasos.append({"header": "Paso 5 — Monotonía: dónde crece y dónde decrece"})

    cortes = list(puntos_criticos) + [p for p in puntos_singulares if p not in puntos_criticos]
    tramos = construir_tramos(f_prime_calc, dominio, cortes)

    if tramos:
        pasos.append({"texto": "Los puntos críticos dividen el dominio en intervalos. En cada uno evaluamos el signo de "
                               "$f'(x)$ en un valor de prueba (positivo: $f$ crece; negativo: $f$ decrece):"})
        filas = ["| Intervalo | Signo de f'(x) | Comportamiento de f |", "|---|---|---|"]
        for t in tramos:
            if t['signo'] == 1:
                sg, comp = "+", "Creciente ↗"
            elif t['signo'] == -1:
                sg, comp = "−", "Decreciente ↘"
            elif t['signo'] == 0:
                sg, comp = "0", "Constante →"
            else:
                sg, comp = "—", "—"
            filas.append(f"| {_fmt_intervalo(t['lo'], t['hi'], t['izq_cerrado'], t['der_cerrado'])} | {sg} | {comp} |")
        pasos.append({"tabla": "\n".join(filas)})

    extremos_locales = []  # (x, y, tipo)
    lineas_clasif = []
    for i in range(len(tramos) - 1):
        t1, t2 = tramos[i], tramos[i + 1]
        if t1['idx'] != t2['idx'] or not (t1['hi'] == t2['lo']):
            continue
        c = t1['hi']
        s1, s2 = t1['signo'], t2['signo']
        if s1 is None or s2 is None:
            continue
        yc = valor_f_exacto(f_orig, c)
        if yc is None:
            continue
        lc = latex_sin_sign(c)
        if s1 > 0 and s2 < 0:
            extremos_locales.append((c, yc, "máximo local"))
            lineas_clasif.append(f"- En $x = {lc}$, $f'$ pasa de $+$ a $-$: **máximo local**, con $f({lc}) = {latex_valor(yc)}$.")
        elif s1 < 0 and s2 > 0:
            extremos_locales.append((c, yc, "mínimo local"))
            lineas_clasif.append(f"- En $x = {lc}$, $f'$ pasa de $-$ a $+$: **mínimo local**, con $f({lc}) = {latex_valor(yc)}$.")
        else:
            lineas_clasif.append(f"- En $x = {lc}$, $f'$ no cambia de signo (la función sigue {'creciendo' if s1 > 0 else 'decreciendo'}): **no es un extremo**.")

    if lineas_clasif:
        pasos.append({"texto": "**Clasificación local** (por el cambio de signo de $f'$):\n\n" + "\n".join(lineas_clasif)})
    elif tramos:
        pasos.append({"texto": "Ningún punto crítico queda en el interior del dominio con $f'$ definida a ambos lados, "
                               "así que no hay extremos locales interiores."})

    # ================================================================
    # PASO 6 — SEGUNDA DERIVADA Y CONCAVIDAD
    # ================================================================
    pasos.append({"header": "Paso 6 — Segunda derivada y concavidad"})
    puntos_inflexion = []
    try:
        if tiene_tramos:
            filas_f2 = [f"{latex_sin_sign(sp.factor(sp.together(sp.diff(d, x))))} & \\text{{si }} {cond}"
                        for cond, _, d in derivadas_est]
            f2_latex = "f''(x) = \\begin{cases} " + " \\\\ ".join(filas_f2) + " \\end{cases}"
        else:
            f2_latex = f"f''(x) = {latex_sin_sign(sp.factor(sp.together(sp.diff(derivadas_est[0][2], x))))}"
    except Exception:
        f2_latex = None

    cortes2 = []
    for c_ in ceros_segunda_derivada(f_calc, dominio) + list(puntos_singulares):
        if not any(c_ == q for q in cortes2):
            cortes2.append(c_)
    try:
        cortes2.sort(key=lambda c: float(sp.N(c)))
    except Exception:
        pass
    tramos2 = construir_tramos(f_prime_calc, dominio, cortes2, signo_fn=_signo_f2(f_calc))

    if tramos2 and any(t['signo'] is not None for t in tramos2) and f2_latex:
        # 6.1 Calcular f''
        pasos.append({"header": "6.1 — Calcular la segunda derivada", "sub": True})
        pasos.append({
            "texto": "Derivamos $f'(x)$ una vez más. El signo de $f''(x)$ dice hacia dónde se curva la gráfica: "
                     "positivo, cóncava hacia arriba (∪); negativo, cóncava hacia abajo (∩).",
            "latex": f2_latex,
        })

        # 6.2 Puntos que dividen la concavidad
        pasos.append({"header": "6.2 — Puntos donde puede cambiar la curvatura", "sub": True})
        if cortes2:
            pasos.append({"texto": "Son los valores donde $f''(x) = 0$ o donde $f''(x)$ no existe:"})
            pasos.append({"latex": ",\\; ".join(f"x = {latex_sin_sign(c)}" for c in cortes2)})
        else:
            pasos.append({"texto": "$f''(x)$ no se anula ni deja de existir en el dominio, así que la curvatura "
                                   "es la misma en todos los intervalos."})

        # 6.3 Tabla de signos de f''
        pasos.append({"header": "6.3 — Tabla de signos de f''(x)", "sub": True})
        pasos.append({"texto": "Evaluamos $f''(x)$ en un valor de prueba de cada intervalo:"})
        filas2 = ["| Intervalo | Signo de f''(x) | Concavidad |", "|---|---|---|"]
        for t in tramos2:
            sg2, cc = {1: ("+", "Cóncava hacia arriba ∪"), -1: ("−", "Cóncava hacia abajo ∩"),
                       0: ("0", "Sin curvatura →")}.get(t['signo'], ("—", "—"))
            filas2.append(f"| {_fmt_intervalo(t['lo'], t['hi'], t['izq_cerrado'], t['der_cerrado'])} | {sg2} | {cc} |")
        pasos.append({"tabla": "\n".join(filas2)})

        # 6.4 Puntos de inflexión
        pasos.append({"header": "6.4 — Puntos de inflexión", "sub": True})
        lineas_infl = []
        for i in range(len(tramos2) - 1):
            t1, t2 = tramos2[i], tramos2[i + 1]
            if t1['idx'] != t2['idx'] or t1['hi'] != t2['lo']:
                continue
            s1, s2_ = t1['signo'], t2['signo']
            if s1 in (None, 0) or s2_ in (None, 0) or s1 == s2_:
                continue
            c = t1['hi']
            yc = valor_f_exacto(f_orig, c)
            if yc is None:
                continue
            puntos_inflexion.append((c, yc))
            lc = latex_sin_sign(c)
            lineas_infl.append(f"- En $x = {lc}$ la concavidad cambia: **punto de inflexión** en $({lc}, {latex_valor(yc)})$.")
        if lineas_infl:
            pasos.append({"texto": "Un punto de inflexión aparece cuando $f''$ cambia de signo:\n\n" + "\n".join(lineas_infl)})
        else:
            pasos.append({"texto": "La concavidad no cambia en ningún punto del dominio: **no hay puntos de inflexión**."})
    else:
        pasos.append({"texto": "No fue posible estudiar el signo de $f''(x)$ en este dominio; "
                               "la clasificación se apoya en la tabla de signos de $f'$ del Paso 5."})

    # 6.5 Criterio de la segunda derivada
    filas_c = ["| Punto crítico | f''(c) | Signo | Conclusión |", "|---|---|---|---|"]
    hay_criterio = False
    for c in puntos_criticos:
        if es_borde_del_dominio(dominio, c):
            continue
        v2 = derivada2_en_punto(f_calc, c)
        if v2 is None:
            continue
        try:
            v2n = float(sp.N(v2))
        except Exception:
            continue
        if v2n > 0:
            sg_c, concl = "+", "Mínimo local ∪"
        elif v2n < 0:
            sg_c, concl = "−", "Máximo local ∩"
        else:
            sg_c, concl = "0", "No concluye (se usa el Paso 5)"
        filas_c.append(f"| x = {texto_valor(c)} | {texto_valor(v2)} | {sg_c} | {concl} |")
        hay_criterio = True
    if hay_criterio:
        pasos.append({"header": "6.5 — Criterio de la segunda derivada", "sub": True})
        pasos.append({"texto": "Evaluamos $f''$ en cada punto estacionario interior: si $f''(c) > 0$ la gráfica "
                               "se curva hacia arriba (mínimo); si $f''(c) < 0$, hacia abajo (máximo)."})
        pasos.append({"tabla": "\n".join(filas_c)})

    # ================================================================
    # PASO 7 — LÍMITES Y ASÍNTOTAS
    # ================================================================
    pasos.append({"header": "Paso 7 — Límites y asíntotas"})

    # Límites en bordes abiertos e infinitos
    limites = []  # (etiqueta_texto, limite, etiqueta_tex)
    pasos_limites = []
    for fr in fronteras:
        if fr['tipo'] == 'cerrado':
            continue
        lim = fr.get('limite')
        if lim is None:
            continue
        val = fr['valor']
        if fr['tipo'] == 'abierto':
            lado_tex = "^{+}" if fr['dir'] == '+' else "^{-}"
            lado_txt = "⁺" if fr['dir'] == '+' else "⁻"
            etq_tex = f"x \\to {latex_sin_sign(val)}{lado_tex}"
            etq_txt = f"x → {texto_valor(val)}{lado_txt}"
            sub = (f"El valor $x = {latex_sin_sign(val)}$ no pertenece al dominio, así que $f$ nunca alcanza este valor; "
                   "solo se acerca a él.")
        else:
            sg = "+\\infty" if val == sp.oo else "-\\infty"
            etq_tex = f"x \\to {sg}"
            etq_txt = "x → +∞" if val == sp.oo else "x → −∞"
            sub = "El infinito no es un punto del dominio: este límite describe hacia dónde se dirige $f$."
        limites.append((etq_txt, lim, etq_tex))
        pasos_limites.append({
            "latex": f"\\lim_{{{etq_tex}}} f(x) = {latex_limite(lim)}",
            "subtext": sub,
        })

    if pasos_limites:
        pasos.append({"texto": "**Límites** en los bordes abiertos y en el infinito:"})
        for pl in pasos_limites:
            pasos.append(pl)
    else:
        pasos.append({"texto": "El dominio no tiene bordes abiertos ni infinitos: no hay límites que calcular."})

    # Asíntotas
    vert, hor, obl = calcular_asintotas(f_calc, fronteras)
    lineas_as = [f"- **Vertical:** $x = {latex_sin_sign(v)}$." for v in vert]
    lineas_as += [f"- **Horizontal:** $y = {latex_sin_sign(l)}$ (cuando $x \\to {'+' if v == sp.oo else '-'}\\infty$)." for v, l in hor]
    lineas_as += [f"- **Oblicua:** $y = {latex_sin_sign(m)}x + {latex_sin_sign(b)}$." for _, m, b in obl]
    pasos.append({"texto": "**Asíntotas:**\n\n" + "\n".join(lineas_as)} if lineas_as
                 else {"texto": "**Asíntotas:** la función no tiene asíntotas verticales, horizontales ni oblicuas."})

    # ================================================================
    # PASO 6 — VEREDICTO FINAL
    # ================================================================
    pasos.append({"header": "Paso 8 — Veredicto final"})

    absoluto_max_existe = bool(candidatos)
    absoluto_min_existe = bool(candidatos)
    max_c = max(candidatos, key=lambda c: _a_float(c['y'])) if candidatos else None
    min_c = min(candidatos, key=lambda c: _a_float(c['y'])) if candidatos else None
    razon_max = None
    razon_min = None
    tol = 1e-9

    if not candidatos:
        razon_max = "no hay ningún punto crítico ni borde donde $f$ alcance un valor máximo."
        razon_min = "no hay ningún punto crítico ni borde donde $f$ alcance un valor mínimo."

    for etq_txt, lim, etq_tex in limites:
        try:
            if lim == sp.oo:
                if absoluto_max_existe or razon_max is None:
                    razon_max = f"cuando ${etq_tex}$, $f(x) \\to +\\infty$: la función no está acotada superiormente."
                absoluto_max_existe = False
            elif lim == -sp.oo:
                if absoluto_min_existe or razon_min is None:
                    razon_min = f"cuando ${etq_tex}$, $f(x) \\to -\\infty$: la función no está acotada inferiormente."
                absoluto_min_existe = False
            elif lim.is_finite is True:
                ln = _a_float(lim)
                if max_c is not None and ln > _a_float(max_c['y']) + tol:
                    if absoluto_max_existe:
                        razon_max = (f"cuando ${etq_tex}$, $f(x) \\to {latex_valor(lim)}$, que supera a todos los "
                                     "valores alcanzados y no se alcanza.")
                    absoluto_max_existe = False
                if min_c is not None and ln < _a_float(min_c['y']) - tol:
                    if absoluto_min_existe:
                        razon_min = (f"cuando ${etq_tex}$, $f(x) \\to {latex_valor(lim)}$, que queda por debajo de todos "
                                     "los valores alcanzados y no se alcanza.")
                    absoluto_min_existe = False
        except Exception:
            pass

    pasos.append({"texto": "Comparamos los valores alcanzados (Paso 4) con los límites (Paso 7):"})

    if absoluto_max_existe and max_c is not None:
        lx = latex_sin_sign(max_c['x'])
        borde_txt = " (en un borde del dominio)" if max_c['tipo'] == 'borde' else ""
        pasos.append({"texto": f"- **Máximo:** el mayor valor alcanzado es $f({lx}) = {latex_valor(max_c['y'])}${borde_txt} "
                               "y ningún límite lo supera, así que es el máximo absoluto."})
    else:
        pasos.append({"texto": f"- **Máximo:** no existe máximo absoluto, porque {razon_max or 'ningún valor domina a todos los demás.'}"})

    if absoluto_min_existe and min_c is not None:
        lx = latex_sin_sign(min_c['x'])
        borde_txt = " (en un borde del dominio)" if min_c['tipo'] == 'borde' else ""
        pasos.append({"texto": f"- **Mínimo:** el menor valor alcanzado es $f({lx}) = {latex_valor(min_c['y'])}${borde_txt} "
                               "y ningún límite queda por debajo, así que es el mínimo absoluto."})
    else:
        pasos.append({"texto": f"- **Mínimo:** no existe mínimo absoluto, porque {razon_min or 'ningún valor queda por debajo de todos los demás.'}"})

    maximos_locales = [e for e in extremos_locales if e[2] == "máximo local"]
    minimos_locales = [e for e in extremos_locales if e[2] == "mínimo local"]

    lineas_finales = []
    if maximos_locales:
        for xv, yv, _ in maximos_locales:
            lineas_finales.append(f"- Máximo local: $f({latex_sin_sign(xv)}) = {latex_valor(yv)}$ en $x = {latex_sin_sign(xv)}$.")
    else:
        lineas_finales.append("- Máximos locales: no hay.")
    if minimos_locales:
        for xv, yv, _ in minimos_locales:
            lineas_finales.append(f"- Mínimo local: $f({latex_sin_sign(xv)}) = {latex_valor(yv)}$ en $x = {latex_sin_sign(xv)}$.")
    else:
        lineas_finales.append("- Mínimos locales: no hay.")

    if absoluto_max_existe and max_c is not None:
        lineas_finales.append(f"- Máximo absoluto: $f({latex_sin_sign(max_c['x'])}) = {latex_valor(max_c['y'])}$ en $x = {latex_sin_sign(max_c['x'])}$.")
    else:
        lineas_finales.append("- Máximo absoluto: no existe.")
    if absoluto_min_existe and min_c is not None:
        lineas_finales.append(f"- Mínimo absoluto: $f({latex_sin_sign(min_c['x'])}) = {latex_valor(min_c['y'])}$ en $x = {latex_sin_sign(min_c['x'])}$.")
    else:
        lineas_finales.append("- Mínimo absoluto: no existe.")

    if puntos_inflexion:
        for xv, yv in puntos_inflexion:
            lineas_finales.append(f"- Punto de inflexión: $({latex_sin_sign(xv)}, {latex_valor(yv)})$.")
    else:
        lineas_finales.append("- Puntos de inflexión: no hay.")

    pasos.append({"texto": "**Respuesta final**\n\n" + "\n".join(lineas_finales)})

    tabla_md = generar_tabla_resumen(
        dominio, puntos_criticos, puntos_singulares,
        [(etq_txt, lim) for etq_txt, lim, _ in limites],
        extremos_locales,
        max_c, min_c, absoluto_max_existe, absoluto_min_existe,
        inflexion=puntos_inflexion
    )
    pasos.append({"texto": "**Tabla resumen**"})
    pasos.append({"tabla": tabla_md})

    if resumen_out is not None:
        resumen_out.update({
            'max_c': max_c, 'min_c': min_c,
            'max_existe': absoluto_max_existe, 'min_existe': absoluto_min_existe,
            'extremos_locales': extremos_locales,
            'inflexion': puntos_inflexion,
            'dominio': dominio,
            'f_orig': f_orig, 'candidatos': candidatos, 'asintotas': (vert, hor, obl),
        })

    return pasos


# ------------------------------------------------------------------
# PROBLEMAS APLICADOS (CON ENUNCIADO)
# ------------------------------------------------------------------
_TRANSF = standard_transformations + (implicit_multiplication_application, convert_xor)


def _numero_o_none(v):
    if v is None or v == "":
        return None
    try:
        return sp.nsimplify(str(v))
    except Exception:
        return None


def _parsear_expresion_en_x(texto):
    expr = sp.sympify(limpiar_sintaxis_matematica(str(texto)), locals={'x': x, 'ln': sp.log})
    if any(sm.name != 'x' for sm in expr.free_symbols):
        raise ValueError("variables extra")
    return expr


def eliminar_variables_sympy(datos):
    """Sustituye las restricciones en el objetivo con SymPy. None si no se puede verificar."""
    try:
        loc = {'x': x, 'ln': sp.log, 'pi': sp.pi}
        obj = parse_expr(str(datos['objetivo_expr']), local_dict=loc, transformations=_TRANSF)
        ecs = []
        for r in datos.get('restricciones') or []:
            izq, der = str(r).split('=')
            ecs.append(parse_expr(izq, local_dict=loc, transformations=_TRANSF)
                       - parse_expr(der, local_dict=loc, transformations=_TRANSF))
        sust = {}
        for e in ecs:
            e = e.subs(sust)
            libres = [s for s in e.free_symbols if s != x]
            if not libres:
                continue
            sol = sp.solve(e, libres[0])
            if len(sol) != 1:
                return None   # ambiguo: se queda con la función de la IA
            sust[libres[0]] = sol[0]
        f = obj
        for _ in range(3):
            f = f.subs(sust)
        f = sp.simplify(f)
        return f if f.free_symbols <= {x} and f.has(x) else None
    except Exception:
        return None


def _hechos_del_optimo(datos, resumen):
    """Valores calculados (con sympy) del punto óptimo, listos para la conclusión."""
    es_max = "max" in str(datos.get("objetivo", "")).lower()
    clave = 'max' if es_max else 'min'
    if not resumen.get(clave + '_existe') or resumen.get(clave + '_c') is None:
        return es_max, None
    cand = resumen[clave + '_c']
    xv, yv = cand['x'], cand['y']
    hechos = []
    desc_x = str(datos.get("descripcion_x", "x"))
    hechos.append(f"{desc_x}: x = {texto_valor(xv)} (≈ {_a_float(xv):.4f})")
    for ov in datos.get("otras_variables") or []:
        try:
            ve = sp.simplify(_parsear_expresion_en_x(ov.get("expresion", "")).subs(x, xv))
            hechos.append(f"{ov.get('nombre', 'medida')} = {texto_valor(ve)} (≈ {_a_float(ve):.4f}) {ov.get('unidad', '')}".strip())
        except Exception:
            continue
    nombre_obj = str(datos.get("nombre_objetivo", "valor óptimo"))
    hechos.append(f"{nombre_obj} {'máximo' if es_max else 'mínimo'} = {texto_valor(yv)} (≈ {_a_float(yv):.4f}) {datos.get('unidad_objetivo', '')}".strip())
    return es_max, hechos


def redactar_conclusion_aplicada(enunciado, datos, resumen):
    es_max, hechos = _hechos_del_optimo(datos, resumen)
    nombre_obj = str(datos.get("nombre_objetivo", "la cantidad pedida"))
    if hechos is None:
        return ("**Conclusión del problema**\n\n"
                f"Con el dominio de este problema no existe un valor {'máximo' if es_max else 'mínimo'} de {nombre_obj}: "
                "la función no lo alcanza dentro de los valores permitidos (revisa los límites del Paso 7).")
    texto_ia = ""
    try:
        usuario = (f"Enunciado:\n{enunciado}\n\nObjetivo: {'maximizar' if es_max else 'minimizar'} {nombre_obj}\n\n"
                   "HECHOS ya calculados:\n" + "\n".join(f"- {h}" for h in hechos))
        texto_ia = llamar_claude(SYSTEM_PROMPT_REDACCION, usuario, 600).strip()
    except Exception:
        texto_ia = ""
    if not texto_ia:
        texto_ia = (f"El valor que {'maximiza' if es_max else 'minimiza'} {nombre_obj} se obtiene con los valores "
                    "siguientes, verificados con el análisis de los pasos anteriores.")
    return "**Conclusión del problema**\n\n" + texto_ia + "\n\n" + "\n".join(f"- {h}" for h in hechos)


def resolver_problema_aplicado(enunciado):
    """
    1) La IA plantea el modelo (función objetivo, restricciones, dominio).
    2) SymPy verifica la sustitución de la restricción.
    3) El motor algebraico resuelve la optimización con los 6 pasos.
    4) La IA redacta la conclusión usando SOLO los números calculados.
    """
    try:
        bruto = llamar_claude(SYSTEM_PROMPT_MODELADO, "Enunciado del problema:\n" + enunciado, 2500)
        datos = extraer_json(bruto)
    except RuntimeError as e:
        if str(e) == "SIN_API":
            raise ErrorProblemaAplicado(
                "Para resolver problemas con enunciado falta configurar la clave ANTHROPIC_API_KEY en los secretos de la app."
            )
        raise ErrorProblemaAplicado("No pude conectar con el servicio de IA. Inténtalo de nuevo en unos segundos.")
    except Exception:
        raise ErrorProblemaAplicado(
            "No pude plantear el problema. Revisa que el enunciado tenga todos los datos y vuelve a escribirlo."
        )

    if not datos.get("es_optimizacion", True):
        raise ErrorProblemaAplicado(
            datos.get("mensaje") or "Este enunciado no parece un problema de optimización de Cálculo 1."
        )

    try:
        f_vis = _parsear_expresion_en_x(datos["funcion"])
    except Exception:
        raise ErrorProblemaAplicado(
            "No logré expresar el problema como una función de una sola variable. Reformula el enunciado con más detalle."
        )

    # Verificación independiente: SymPy hace la sustitución de la restricción.
    f_ver = eliminar_variables_sympy(datos)
    nota_sympy = ""
    if f_ver is not None:
        f_vis = f_ver
        nota_sympy = ("\n\n**Sustitución verificada:** al reemplazar la restricción en la función objetivo "
                      f"queda $f(x) = {latex_sin_sign(f_ver)}$.")

    dom = datos.get("dominio") or {}
    mn, mx = _numero_o_none(dom.get("min")), _numero_o_none(dom.get("max"))
    intervalo = None
    if mn is not None or mx is not None:
        a = mn if mn is not None else -sp.oo
        b = mx if mx is not None else sp.oo
        lo = True if a == -sp.oo else (not bool(dom.get("min_incluido", False)))
        ro = True if b == sp.oo else (not bool(dom.get("max_incluido", False)))
        if not bool(a < b):
            raise ErrorProblemaAplicado("El dominio que se obtuvo del enunciado no es válido. Reformula el problema.")
        intervalo = (a, b, lo, ro)

    f_expr = hacer_reales_potencias_fraccionarias(f_vis)
    resumen = {}
    try:
        pasos = construir_pasos_narrativos(f_expr, intervalo, f_expr_visual=f_vis, resumen_out=resumen)
    except Exception:
        raise ErrorProblemaAplicado("No pude resolver la función obtenida del enunciado. Reformula el problema.")

    objetivo = str(datos.get("objetivo", "optimizar")).lower()
    nombre_obj = html.escape(str(datos.get("nombre_objetivo", "la cantidad pedida")))
    desc_x = html.escape(str(datos.get("descripcion_x", "la variable del problema")))
    tips = [
        ("Paso 1 — Entender el problema",
         f"Identifica qué debes {html.escape(objetivo)}: {nombre_obj}. Dibuja la situación y anota los datos con sus unidades."),
        ("Paso 2 — Plantear el modelo",
         f"Escribe {nombre_obj} como una fórmula y usa la condición del problema para dejarla con una sola variable. Aquí x representa: {desc_x}."),
        ("Paso 3 — Dominio con sentido",
         "Las medidas físicas deben ser positivas y respetar los límites del enunciado; ese dominio decide si los bordes cuentan."),
        ("Paso 4 — Optimizar",
         "Deriva, halla los puntos críticos y decide con la tabla de signos (y con f'') si es máximo o mínimo."),
        ("Paso 5 — Responder",
         "Regresa al contexto: calcula las demás medidas, escribe las unidades y comprueba que el resultado tenga sentido."),
    ]

    conclusion = redactar_conclusion_aplicada(enunciado, datos, resumen)
    resumen_txt = enunciado.strip().replace("\n", " ")
    titulo = "📐 " + (resumen_txt[:26].rstrip() + "..." if len(resumen_txt) > 26 else resumen_txt)

    return {
        "tipo": "aplicado",
        "enunciado": enunciado,
        "funcion_original": enunciado,
        "titulo_bonito": titulo,
        "funcion_latex": latex_sin_sign(f_vis),
        "intervalo_usuario": intervalo,
        "planteamiento_md": str(datos.get("planteamiento", "")) + nota_sympy,
        "conclusion_md": conclusion,
        "tips": tips,
        "pasos_narrativos": pasos,
        "ambiguedad_fraccion": None,
        "resumen": resumen,
    }


# --- INTERFAZ PRINCIPAL DE CONVERSACIÓN ---
if st.session_state.problema_activo:
    prob = st.session_state.problema_activo

    with st.chat_message("user", avatar="👤"):
        funcion_original = prob.get('funcion_original', '')
        if prob.get('tipo') == 'aplicado':
            st.markdown(
                '<div class="mirror-line">Problema de optimización</div>',
                unsafe_allow_html=True
            )
            st.markdown(post_procesar_texto(prob.get('enunciado', '')))
            st.markdown("**Función a optimizar (en una sola variable):**")
            st.latex(f"f(x) = {prob['funcion_latex']}")
            if prob.get('intervalo_usuario'):
                st.markdown(f"Dominio del problema: x ∈ {texto_intervalo_usuario(prob['intervalo_usuario'])}")
        else:
            st.markdown(
                f'<div class="mirror-line">Función analizada: {html.escape(funcion_original)}</div>',
                unsafe_allow_html=True
            )
            st.markdown("**Función interpretada:**")
            st.latex(f"f(x) = {prob['funcion_latex']}")
            if prob.get('intervalo_usuario'):
                st.markdown(f"Intervalo indicado: x ∈ {texto_intervalo_usuario(prob['intervalo_usuario'])}")
        
    with st.chat_message("assistant", avatar="🧠"):
        st.markdown(
            """
            <div class="tips-intro">
                <div class="tips-intro-title">Orientaciones previas al análisis</div>
                <div>Antes de desarrollar la solución, revisaremos brevemente los aspectos clave de la función y las condiciones que intervienen en su optimización.</div>
            </div>
            """,
            unsafe_allow_html=True
        )

        amb = prob.get('ambiguedad_fraccion')
        if amb:
            denominador, signo, resto = amb
            st.markdown(
                f'<div class="aviso-box-chat">⚠️ <strong>Revisa tu fracción:</strong> '
                f'como escribiste la división seguida de "{signo} {resto}" sin paréntesis, la '
                f'interpretamos siguiendo el orden estándar de operaciones matemáticas: solo '
                f'<strong>{denominador}</strong> queda en el denominador, y <strong>{signo} {resto}</strong> '
                f'se suma o resta por fuera de la fracción (revisa la función mostrada arriba). '
                f'Si en realidad querías que <strong>{denominador} {signo} {resto}</strong> fuera el '
                f'denominador completo, vuelve a escribirla usando paréntesis, por ejemplo: '
                f'<code>.../({denominador} {signo} {resto})</code>.</div>',
                unsafe_allow_html=True
            )

        for titulo_tip, contenido_tip in prob.get('tips', []):
            st.markdown(
                f'<div class="tip-box-chat">💡 <strong>{post_procesar_texto(titulo_tip)}:</strong> {post_procesar_texto(contenido_tip)}</div>',
                unsafe_allow_html=True
            )

        st.markdown("Cuando estés listo/a, revela la solución completa paso a paso 👇")
        
        if not st.session_state.mostrar_solucion:
            if st.button("🔍 Revelar Solución Paso a Paso"):
                st.session_state.mostrar_solucion = True
                st.rerun()
        else:
            st.markdown("---")
            st.markdown(
                """
                <div class="solution-intro">
                    <div class="solution-title">🧩 Demostración completa de la optimización</div>
                    <div class="solution-subtitle">Desarrollo ordenado en 8 pasos, desde el dominio hasta el veredicto final.</div>
                </div>
                """,
                unsafe_allow_html=True
            )
            if prob.get('tipo') == 'aplicado':
                st.markdown("**Planteamiento del problema**")
                st.markdown(post_procesar_texto(prob.get('planteamiento_md', '')))
                st.markdown("Con el modelo planteado, resolvemos la optimización en una sola variable:")
            else:
                st.markdown("Vamos a resolverla como optimización en una sola variable:")
            st.latex(f"f(x) = {prob['funcion_latex']}")

            primer_paso = True
            for paso in prob['pasos_narrativos']:
                if paso.get('header'):
                    if not primer_paso and not paso.get('sub'):
                        st.markdown('<div class="step-divider"></div>', unsafe_allow_html=True)
                    primer_paso = False
                    st.markdown(
                        f'<div class="paso-header">{post_procesar_texto(paso["header"])}</div>',
                        unsafe_allow_html=True
                    )
                if paso.get('texto'):
                    st.markdown(post_procesar_texto(paso['texto']))
                if paso.get('latex'):
                    st.latex(post_procesar_latex(paso['latex']))
                if paso.get('subtext'):
                    subtext_html = post_procesar_texto(paso["subtext"]).replace("\n", "<br>")
                    st.markdown(f'<div class="paso-subtext">{subtext_html}</div>', unsafe_allow_html=True)
                if paso.get('tabla'):
                    st.markdown(post_procesar_texto(paso['tabla']))

            if prob.get('tipo') == 'aplicado' and prob.get('conclusion_md'):
                st.markdown('<div class="step-divider"></div>', unsafe_allow_html=True)
                st.markdown(post_procesar_texto(prob['conclusion_md']))

            # Gráfica de la función con extremos, inflexiones y asíntotas
            res = prob.get('resumen')
            if res and res.get('f_orig') is not None:
                try:
                    import matplotlib.pyplot as plt
                    fig = graficar_resumen(res)
                    st.markdown("**Gráfica de la función**")
                    st.pyplot(fig)
                    plt.close(fig)
                except Exception:
                    pass

            if st.button("Ocultar solución"):
                st.session_state.mostrar_solucion = False
                st.rerun()
                
        st.markdown("<br>", unsafe_allow_html=True)
        if st.button("🔄 Iniciar otro análisis"):
            st.session_state.problema_activo = None
            st.session_state.mostrar_solucion = False
            st.rerun()

else:
    import streamlit.components.v1 as components

    with st.chat_message("assistant", avatar="🧠"):
        st.markdown(
            """
            <div class="welcome-box">
                <div class="welcome-title">Bienvenido/a 👋</div>
                <p class="welcome-text">
                    Te muestro, paso a paso y de forma sencilla, dónde una función alcanza su valor más alto y más bajo.
                </p>
                <p class="welcome-instruction">
                    <strong>Agrega tu función f(x) en una sola variable.</strong>
                </p>
            </div>
            """,
            unsafe_allow_html=True
        )
    
    # Botón de calculadora: popover nativo de Streamlit junto al botón de enviar.
    st.markdown("""
    <style>
        .st-key-teclado_boton {
            position: fixed !important;
            z-index: 2147483647 !important;
            display: block !important;
            visibility: visible !important;
            opacity: 1 !important;
            pointer-events: auto !important;
            margin: 0 !important;
            padding: 0 !important;
        }
        .st-key-teclado_boton > div,
        .st-key-teclado_boton [data-testid="stElementContainer"],
        .st-key-teclado_boton [data-testid="stVerticalBlock"],
        .st-key-teclado_boton [data-testid="stVerticalBlockBorderWrapper"] {
            width: 100% !important;
            height: 100% !important;
            margin: 0 !important;
            padding: 0 !important;
            box-sizing: border-box !important;
        }
        .st-key-teclado_boton button {
            width: 100% !important;
            height: 100% !important;
            min-width: 0 !important;
            min-height: 0 !important;
            margin: 0 !important;
            padding: 0 !important;
            box-sizing: border-box !important;
            display: flex !important;
            align-items: center !important;
            justify-content: center !important;
            visibility: visible !important;
            opacity: 1 !important;
            cursor: pointer !important;
            font-size: 21px !important;
            line-height: 1 !important;
            color: #ffffff !important;
            background: #007A33 !important;
            border: 1px solid #005E27 !important;
            border-radius: 10px !important;
            box-shadow: 0 4px 10px rgba(0,122,51,.28) !important;
            overflow: hidden !important;
            position: relative !important;
        }
        .st-key-teclado_boton button:hover,
        .st-key-teclado_boton button:focus,
        .st-key-teclado_boton button:focus-visible,
        .st-key-teclado_boton button:active {
            background: #007A33 !important;
            color: #ffffff !important;
            border-color: #005E27 !important;
            box-shadow: 0 4px 10px rgba(0,122,51,.28) !important;
            outline: none !important;
        }
        .st-key-teclado_boton button::before {
            content: "" !important;
            display: block !important;
            width: 11px !important;
            height: 11px !important;
            border: 2px solid #ffffff !important;
            border-radius: 50% !important;
            position: absolute !important;
            left: 50% !important;
            top: 50% !important;
            transform: translate(-65%, -65%) !important;
            box-sizing: border-box !important;
        }
        .st-key-teclado_boton button::after {
            content: "" !important;
            display: block !important;
            width: 7px !important;
            height: 2px !important;
            background: #ffffff !important;
            border-radius: 2px !important;
            position: absolute !important;
            left: calc(50% + 4px) !important;
            top: calc(50% + 4px) !important;
            transform: rotate(45deg) !important;
            transform-origin: left center !important;
        }
        .st-key-teclado_boton button > * {
            display: block !important;
            visibility: visible !important;
            opacity: 1 !important;
        }
        .st-key-teclado_boton button svg {
            width: 22px !important;
            height: 22px !important;
            display: block !important;
            color: #ffffff !important;
            visibility: visible !important;
            opacity: 1 !important;
        }
    </style>
    """, unsafe_allow_html=True)

    with st.popover("", help="Abrir teclado matemático", key="teclado_boton"):
        st.markdown('<div class="math-title">⌨ Teclado matemático</div>', unsafe_allow_html=True)
        components.html("""
        <style>
            * { box-sizing: border-box; }
            body { margin:0; font-family:Arial,sans-serif; background:#10141d; color:#e8eef5; }
            .keyboard { background:linear-gradient(145deg,#151b26,#0e121a); border:1px solid #273241; border-radius:12px; padding:10px; box-shadow:0 8px 24px rgba(0,0,0,.35); }
            .tabs { display:flex; gap:5px; margin-bottom:9px; }
            .tab { flex:1; border:1px solid #303b4a; background:#1b2330; color:#cbd5e1; padding:7px 9px; border-radius:7px; cursor:pointer; font-weight:700; }
            .tab.active { background:#087d35; border-color:#12a64d; color:#fff; }
            .panel { display:none; }
            .panel.active { display:block; }
            .row { display:flex; gap:5px; margin:5px 0; }
            button.key { flex:1; min-width:0; height:36px; border:1px solid #303b4a; border-radius:7px; background:#202936; cursor:pointer; font-size:15px; font-weight:600; color:#f1f5f9; box-shadow:inset 0 1px 0 rgba(255,255,255,.04); }
            button.key:hover { background:#293443; border-color:#3caa68; }
            button.key:active { transform:scale(.97); background:#0a7131; }
            .wide { flex:2 !important; }
            .hint { font-size:10px; color:#8996a7; margin:7px 2px 1px; }
            .math-title { color:#dfe8f0; font-weight:700; margin-bottom:7px; }
            .fraction-row { justify-content:center; }
            button.key.fraction-key {
                flex: 0 0 76px;
                width: 76px;
                height: 42px;
                position: relative;
                display:flex;
                flex-direction:column;
                align-items:center;
                justify-content:center;
                gap:1px;
                font-size:14px;
                font-weight:700;
                line-height:1;
                background:linear-gradient(145deg,#273244,#18202c);
                border:1px solid #526176;
                box-shadow:0 4px 12px rgba(0,0,0,.22), inset 0 1px 0 rgba(255,255,255,.06);
            }
            button.key.fraction-key:hover {
                background:linear-gradient(145deg,#334158,#202b3b);
                border-color:#59c985;
            }
            .fraction-key .frac-top,
            .fraction-key .frac-bottom { display:block; line-height:11px; }
            .fraction-key .frac-line {
                display:block;
                width:28px;
                height:2px;
                border-radius:2px;
                background:#f1f5f9;
                margin:1px 0;
            }
        </style>
        <div class="keyboard">
        <div class="tabs">
          <button class="tab active" data-tab="num">123</button>
          <button class="tab" data-tab="fx">f(x)</button>
          <button class="tab" data-tab="sym">#&amp;¬</button>
        </div>

        <div id="num" class="panel active">
          <div class="row"><button class="key" data-v="7">7</button><button class="key" data-v="8">8</button><button class="key" data-v="9">9</button><button class="key" data-v="*">×</button><button class="key" data-v="/">÷</button></div>
          <div class="row"><button class="key" data-v="4">4</button><button class="key" data-v="5">5</button><button class="key" data-v="6">6</button><button class="key" data-v="+">+</button><button class="key" data-v="-">−</button></div>
          <div class="row"><button class="key" data-v="1">1</button><button class="key" data-v="2">2</button><button class="key" data-v="3">3</button><button class="key" data-v="(">(</button><button class="key" data-v=")">)</button></div>
          <div class="row"><button class="key" data-v="0">0</button><button class="key" data-v=".">.</button><button class="key" data-action="backspace">⌫</button><button class="key wide" data-action="clear">AC</button></div>
        </div>

        <div id="sym" class="panel">
      <div class="row"><button class="key" data-v="<">&lt;</button><button class="key" data-v=">">&gt;</button><button class="key" data-v="≤">≤</button><button class="key" data-v="≥">≥</button><button class="key" data-v="=">=</button></div>
      <div class="row"><button class="key" data-v="[">[</button><button class="key" data-v="]">]</button><button class="key" data-v="(">(</button><button class="key" data-v=")">)</button><button class="key" data-v=",">,</button></div>
    </div>
    <div id="fx" class="panel">
          <div class="row"><button class="key" data-v="x">x</button><button class="key" data-v="pi">π</button><button class="key" data-v="E">e</button></div>
          <div class="row"><button class="key" data-v="**2">x²</button><button class="key" data-v="**">xⁿ</button><button class="key" data-v="sqrt(">√(</button></div>
          <div class="row"><button class="key" data-v="sin(">sin(</button><button class="key" data-v="cos(">cos(</button><button class="key" data-v="tan(">tan(</button></div>
          <div class="row"><button class="key" data-v="ln(">ln(</button><button class="key" data-v="log(">log(</button><button class="key" data-v="exp(">exp(</button></div>
          <div class="row"><button class="key" data-v="Abs(">|x|</button><button class="key" data-v="**(-1)">x⁻¹</button></div>
          <div class="row fraction-row"><button class="key fraction-key" data-action="fraction"><span class="frac-top">a</span><span class="frac-line"></span><span class="frac-bottom">b</span></button></div></div>
        </div>

        <div class="row"><button class="key" data-action="left">←</button><button class="key" data-action="right">→</button><button class="key wide" data-action="home">Inicio</button><button class="key wide" data-action="end">Fin</button></div>
        <div class="hint">Las teclas se insertan directamente en la barra de función.</div>
        </div>

        <script>
        (function() {
          function getInput() {
            const doc = window.parent.document;
            return doc.querySelector('textarea[aria-label*="Ingresa la función"]') ||
                   doc.querySelector('textarea[placeholder*="Ingresa la función"]') ||
                   doc.querySelector('[data-testid="stChatInput"] textarea') ||
                   doc.querySelector('textarea');
          }

          function setValue(el, value, start, end) {
            const setter = Object.getOwnPropertyDescriptor(HTMLTextAreaElement.prototype, 'value').set;
            setter.call(el, value);
            el.dispatchEvent(new Event('input', {bubbles:true}));
            el.dispatchEvent(new Event('change', {bubbles:true}));
            try { el.setSelectionRange(start, end); } catch(e) {}
            el.focus();
          }

          function insert(value) {
            const el = getInput();
            if (!el) return;
            const start = typeof el.selectionStart === 'number' ? el.selectionStart : el.value.length;
            const end = typeof el.selectionEnd === 'number' ? el.selectionEnd : start;
            const next = el.value.slice(0,start) + value + el.value.slice(end);
            setValue(el, next, start + value.length, start + value.length);
          }

          function backspace() {
            const el = getInput();
            if (!el) return;
            let start = el.selectionStart || 0, end = el.selectionEnd || 0;
            if (start === end && start > 0) start--;
            const next = el.value.slice(0,start) + el.value.slice(end);
            setValue(el, next, start, start);
          }

          function clearInput() {
            const el = getInput();
            if (!el) return;
            setValue(el, '', 0, 0);
          }

          function moveCursor(direction) {
            const el = getInput();
            if (!el) return;
            let pos = typeof el.selectionStart === 'number' ? el.selectionStart : el.value.length;
            if (direction === 'left') pos = Math.max(0, pos - 1);
            if (direction === 'right') pos = Math.min(el.value.length, pos + 1);
            if (direction === 'home') pos = 0;
            if (direction === 'end') pos = el.value.length;
            el.focus();
            try { el.setSelectionRange(pos, pos); } catch(e) {}
          }

          function fraction() {
            const el = getInput();
            if (!el) return;
            const start = typeof el.selectionStart === 'number' ? el.selectionStart : el.value.length;
            const end = typeof el.selectionEnd === 'number' ? el.selectionEnd : start;
            const selected = el.value.slice(start, end);
            const value = selected ? '(' + selected + ')/()' : '()/()';
            const cursor = selected ? start + selected.length + 4 : start + 1;
            const next = el.value.slice(0,start) + value + el.value.slice(end);
            setValue(el, next, cursor, cursor);
          }

          document.querySelectorAll('.tab').forEach(tab => {
            tab.addEventListener('click', () => {
              document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
              document.querySelectorAll('.panel').forEach(p => p.classList.remove('active'));
              tab.classList.add('active');
              document.getElementById(tab.dataset.tab).classList.add('active');
            });
          });

          document.querySelectorAll('button.key').forEach(btn => {
            btn.addEventListener('click', () => {
              const action = btn.dataset.action;
              if (action === 'backspace') backspace();
              else if (action === 'clear') clearInput();
              else if (action === 'left') moveCursor('left');
              else if (action === 'right') moveCursor('right');
              else if (action === 'home') moveCursor('home');
              else if (action === 'end') moveCursor('end');
              else if (action === 'fraction') fraction();
              else if (btn.dataset.v !== undefined) insert(btn.dataset.v);
            });
          });
        })();
        </script>
        """, height=315, scrolling=False)

    # Alineación dinámica: coloca la calculadora junto al botón de enviar.
    components.html("""
    <script>
    (function() {
        function alinear() {
            const doc = window.parent.document;
            const chat = doc.querySelector('[data-testid="stChatInput"]');
            const wrapper = doc.querySelector('.st-key-teclado_boton');
            if (!chat || !wrapper) return;

            const botones = chat.querySelectorAll('button');
            if (!botones.length) return;
            const enviar = botones[botones.length - 1];
            const r = enviar.getBoundingClientRect();
            if (r.width < 2 || r.height < 2) return;

            const gap = 6;
            wrapper.style.setProperty('position', 'fixed', 'important');
            wrapper.style.setProperty('display', 'block', 'important');
            wrapper.style.setProperty('visibility', 'visible', 'important');
            wrapper.style.setProperty('opacity', '1', 'important');
            wrapper.style.setProperty('z-index', '2147483647', 'important');
            wrapper.style.setProperty('top', r.top + 'px', 'important');
            wrapper.style.setProperty('left', (r.left - r.width - gap) + 'px', 'important');
            wrapper.style.setProperty('width', r.width + 'px', 'important');
            wrapper.style.setProperty('height', r.height + 'px', 'important');

            const trigger = wrapper.querySelector('button');
            if (trigger) {
                const cs = window.parent.getComputedStyle(enviar);
                trigger.style.setProperty('width', '100%', 'important');
                trigger.style.setProperty('height', '100%', 'important');
                trigger.style.setProperty('background', cs.backgroundColor || '#007A33', 'important');
                trigger.style.setProperty('border', cs.border || '1px solid #005E27', 'important');
                trigger.style.setProperty('border-radius', cs.borderRadius || '10px', 'important');
                trigger.style.setProperty('box-shadow', cs.boxShadow || '0 4px 10px rgba(0,0,0,.2)', 'important');
                trigger.style.setProperty('color', cs.color || '#fff', 'important');
                trigger.style.setProperty('visibility', 'visible', 'important');
                trigger.style.setProperty('opacity', '1', 'important');
            }
        }

        alinear();
        window.parent.addEventListener('resize', alinear);
        window.parent.addEventListener('scroll', alinear, true);
        setInterval(alinear, 300);
    })();
    </script>
    """, height=0)

    user_input = st.chat_input("Agrega tu función f(x) en una sola variable...")
    
    if user_input and not es_funcion_pura(user_input):
        # Problema con enunciado: la IA lo plantea, SymPy verifica, el motor lo resuelve.
        if len(user_input.split()) < 6:
            st.warning("Agrega una función f(x) en una sola variable, por ejemplo: x^3 - 3x")
        else:
            try:
                with st.spinner("Planteando y resolviendo el problema..."):
                    nuevo_item = resolver_problema_aplicado(user_input)
                st.session_state.historial_problemas.append(nuevo_item)
                st.session_state.problema_activo = nuevo_item
                st.session_state.mostrar_solucion = False
                st.rerun()
            except ErrorProblemaAplicado as e:
                st.info(str(e))
            except Exception:
                st.error("⚠️ No pude resolver este problema. Revisa el enunciado e inténtalo de nuevo.")
    elif user_input:
        if "x" not in user_input.lower():
            st.warning(
                "Por favor, ingresa una función matemática de una sola variable en términos de x."
            )
        else:
            funcion_original = user_input
            funcion_str = user_input

            intervalo_usuario = None
            resultado_intervalo = extraer_intervalo_usuario(funcion_str)
            if resultado_intervalo is not None:
                a_usr, b_usr, funcion_str_sin_intervalo = resultado_intervalo
                intervalo_usuario = (a_usr, b_usr)
                funcion_str_para_parsear = funcion_str_sin_intervalo
            else:
                funcion_str_para_parsear = funcion_str

            try:
                funcion_saneada = limpiar_sintaxis_matematica(funcion_str_para_parsear)

                # Expresión visual: lo que escribió el estudiante, interpretado por SymPy.
                f_expr_visual = sp.sympify(
                    funcion_saneada, locals={'x': x, 'ln': sp.log}
                )

                # Expresión interna con corrección real de potencias fraccionarias.
                f_expr = hacer_reales_potencias_fraccionarias(f_expr_visual)

                if not any(s.name == 'x' for s in f_expr_visual.free_symbols):
                    st.error("⚠️ La expresión no contiene la variable 'x'. Inténtalo de nuevo.")
                else:
                    tips = generar_tips(f_expr_visual, intervalo_usuario)
                    resumen = {}
                    pasos_narrativos = construir_pasos_narrativos(
                        f_expr,
                        intervalo_usuario,
                        f_expr_visual=f_expr_visual,
                        resumen_out=resumen
                    )

                    funcion_interpretada_latex = latex_sin_sign(f_expr_visual)
                    titulo_bonito_str = formatear_titulo_bonito(f_expr_visual)
                    ambiguedad_fraccion = detectar_posible_ambiguedad_fraccion(funcion_str_para_parsear)

                    nuevo_item = {
                        "funcion_original": funcion_original,
                        "titulo_bonito": titulo_bonito_str,
                        "funcion_latex": funcion_interpretada_latex,
                        "intervalo_usuario": intervalo_usuario,
                        "tips": tips,
                        "pasos_narrativos": pasos_narrativos,
                        "ambiguedad_fraccion": ambiguedad_fraccion,
                        "resumen": resumen,
                    }
                    st.session_state.historial_problemas.append(nuevo_item)
                    st.session_state.problema_activo = nuevo_item
                    st.session_state.mostrar_solucion = False
                    st.rerun()
                    
            except Exception as e:
                st.error(f"⚠️ No pude interpretar la sintaxis matemática. Asegúrate de ingresar una expresión válida en términos de x. Detalle: {e}")
