# conversion_app
import streamlit as st

"""
Bidirectional conversion calculator for common oil‑field / fluid‑handling additive
units.  Enter either the **input concentration** or the **desired output amount**
for the chosen target volume and the other value is automatically computed.

To deploy online:
  • Sign in at https://share.streamlit.io (or push to any Streamlit‑friendly host)
  • Create a new app pointing at this file
  • Done – the UI is generated automatically.
"""

st.set_page_config(page_title="Additive Conversion Calculator", page_icon="🧪", layout="centered")

st.title("🧪 Additive Conversion Calculator")

# --- User‑adjustable total fluid volume ------------------------------------
volume_ml = st.number_input(
    "Target fluid volume (mL)", min_value=1.0, value=2000.0, step=1.0, format="%g"
)
volume_l = volume_ml / 1000  # for mol/L calculations

st.markdown("---")

# --- Conversion selection --------------------------------------------------
conversion = st.selectbox(
    "Select conversion type",
    (
        "lbs / gal  ↔  g / volume",
        "gal / bbl  ↔  mL / volume",
        "gal / 1000 gal  ↔  mL / volume",
        "mol / L  ↔  g / volume (requires molar mass)",
    ),
)

# Helper constants
LB_TO_G = 453.59237
GAL_TO_ML = 3785.411784
BBL_TO_GAL = 42

# --- Calculator functions --------------------------------------------------

def lbs_per_gal_to_g(lbs_per_gal: float) -> float:
    """Forward: lbs/gal → grams for the selected volume"""
    grams_per_ml = (lbs_per_gal * LB_TO_G) / GAL_TO_ML
    return grams_per_ml * volume_ml


def g_to_lbs_per_gal(grams: float) -> float:
    """Reverse: grams → lbs/gal required to hit that mass at chosen volume"""
    grams_per_ml = grams / volume_ml
    return grams_per_ml * GAL_TO_ML / LB_TO_G


def gal_per_bbl_to_ml(gal_per_bbl: float) -> float:
    ml_per_ml = (gal_per_bbl / BBL_TO_GAL)  # fraction by volume
    return ml_per_ml * volume_ml


def ml_to_gal_per_bbl(ml_: float) -> float:
    ml_per_ml = ml_ / volume_ml
    return ml_per_ml * BBL_TO_GAL


def gal_per_thousand_to_ml(gal_per_thousand: float) -> float:
    return gal_per_thousand / 1000 * volume_ml


def ml_to_gal_per_thousand(ml_: float) -> float:
    return ml_ * 1000 / volume_ml


def molar_to_grams(mol_per_l: float, molar_mass: float) -> float:
    return mol_per_l * molar_mass * volume_l


def grams_to_molar(grams: float, molar_mass: float) -> float:
    return grams / (molar_mass * volume_l)

# --- UI per conversion -----------------------------------------------------

if conversion.startswith("lbs"):
    col1, col2 = st.columns(2)
    with col1:
        input_val = st.number_input("Input concentration (lbs / gal)", value=0.0, format="%g")
    with col2:
        desired_output = st.number_input("Desired output (g)", value=0.0, format="%g")

    if input_val:
        st.success(f"Output: **{lbs_per_gal_to_g(input_val):,.4g} g** for {volume_ml:,.0f} mL")
    if desired_output:
        st.info(f"Required input: **{g_to_lbs_per_gal(desired_output):,.4g} lbs/gal**")

elif conversion.startswith("gal / bbl"):
    col1, col2 = st.columns(2)
    with col1:
        input_val = st.number_input("Input concentration (gal / bbl)", value=0.0, format="%g")
    with col2:
        desired_output = st.number_input("Desired output (mL)", value=0.0, format="%g")

    if input_val:
        st.success(f"Output: **{gal_per_bbl_to_ml(input_val):,.4g} mL** for {volume_ml:,.0f} mL")
    if desired_output:
        st.info(f"Required input: **{ml_to_gal_per_bbl(desired_output):,.4g} gal/bbl**")

elif conversion.startswith("gal / 1000"):
    col1, col2 = st.columns(2)
    with col1:
        input_val = st.number_input("Input concentration (gal / 1000 gal)", value=0.0, format="%g")
    with col2:
        desired_output = st.number_input("Desired output (mL)", value=0.0, format="%g")

    if input_val:
        st.success(f"Output: **{gal_per_thousand_to_ml(input_val):,.4g} mL** for {volume_ml:,.0f} mL")
    if desired_output:
        st.info(f"Required input: **{ml_to_gal_per_thousand(desired_output):,.4g} gal/1000 gal**")

else:  # mol / L ↔ g
    molar_mass = st.number_input("Molar mass of substance (g / mol)", min_value=0.0, value=58.44, format="%g")
    col1, col2 = st.columns(2)
    with col1:
        input_val = st.number_input("Input concentration (mol / L)", value=0.0, format="%g")
    with col2:
        desired_output = st.number_input("Desired output (g)", value=0.0, format="%g")

    if molar_mass <= 0:
        st.warning("Enter a valid molar mass (g / mol) > 0")
    else:
        if input_val:
            st.success(f"Output: **{molar_to_grams(input_val, molar_mass):,.4g} g** for {volume_ml:,.0f} mL")
        if desired_output:
            st.info(f"Required input: **{grams_to_molar(desired_output, molar_mass):,.4g} mol / L**")

st.markdown("---")

st.caption(
    "Developed with ❤️ using Streamlit.  Constants: 1 lb = 453.59237 g; "
    "1 US gal = 3 785.411784 mL; 1 bbl = 42 gal."
)
