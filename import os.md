import os  
import streamlit as st  
import pandas as pd  
from pydantic import BaseModel, Field  
from typing import List, Optional  
from google import genai  
from google.genai import types  
from PIL import Image  
  
# Set professional layout configuration  
st.set_page_config(  
    page_title="BrandPulse AI - Senior UX Auditor",  
    page_icon="🎨",  
    layout="wide",  
    initial_sidebar_state="expanded"  
)  
  
# --- 1. Define the Structured Pydantic Output Schemas ---  
class Readability(BaseModel):  
    status: str  
    assessment: str  
  
class VisualHierarchyAndTypography(BaseModel):  
    score: int = Field(ge=0, le=100)  
    primaryHeaderLegibility: str  
    fontPairingCritique: str  
    readability: Readability  
    recommendation: str  
  
class ContrastRatios(BaseModel):  
    wcagComplianceStatus: str  
    primaryTextOnBackground: str  
    ctaButtonOnBackground: str  
  
class ColorHarmony(BaseModel):  
    paletteType: str  
    assessment: str  
  
class ColorPaletteAndBrandConsistency(BaseModel):  
    score: int = Field(ge=0, le=100)  
    dominantBrandColors: List[str]  
    colorHarmony: ColorHarmony  
    contrastRatios: ContrastRatios  
    recommendation: str  
  
class CtaVisibility(BaseModel):  
    salienceScore: int = Field(ge=0, le=100)  
    critique: str  
  
class LowContrastElement(BaseModel):  
    element: str  
    estimatedRatio: str  
    suggestedFix: str  
  
class UxAndAccessibility(BaseModel):  
    score: int = Field(ge=0, le=100)  
    wcagComplianceTier: str  
    ctaVisibility: CtaVisibility  
    lowContrastElements: List[LowContrastElement]  
    recommendation: str  
  
class GrowthRoadmapItem(BaseModel):  
    priority: str  
    title: str  
    impactArea: str  
    effort: str  
    recommendation: str  
  
class Hotspot(BaseModel):  
    id: str  
    x: float  
    y: float  
    title: str  
    severity: str  
    pillar: str  
    comment: str  
    suggestedFix: str  
  
class Pillars(BaseModel):  
    visualHierarchyAndTypography: VisualHierarchyAndTypography  
    colorPaletteAndBrandConsistency: ColorPaletteAndBrandConsistency  
    uxAndAccessibility: UxAndAccessibility  
  
class BrandAssetAudit(BaseModel):  
    overallHealthScore: int = Field(ge=0, le=100)  
    brandMaturityLevel: str  
    executiveSummary: str  
    businessGrowthImpact: str  
    pillars: Pillars  
    hotspots: List[Hotspot]  
    actionableGrowthRoadmap: List[GrowthRoadmapItem]  
  
# --- 2. Audit Execution Logic ---  
def run_brandpulse_audit(uploaded_file, api_key: str, brand_context: Optional[str] = None) -> BrandAssetAudit:  
    client = genai.Client(api_key=api_key)  
    image = Image.open(uploaded_file)  
  
    system_instruction = (  
        "You are an elite Senior Design Strategist, Creative Director, and Accessibility Auditor. "  
        "Analyze the provided brand artifact across three core pillars: "  
        "1) Visual Hierarchy & Typography, 2) Color Palette & Brand Consistency, 3) UX & Accessibility (WCAG). "  
        "Calculate contrast ratios, inspect CTA salience, and identify exact normalized coordinate hotspots (x: 0-100%, y: 0-100%)."  
    )  
  
    prompt_text = (  
        "Perform an exhaustive diagnostic brand and UX audit on this visual artifact. "  
        f"Additional context: {brand_context or 'Standard B2B / Consumer digital product'}. "  
        "Provide specific, highly actionable recommendations for conversion and accessibility."  
    )  
  
    response = client.models.generate_content(  
        model="gemini-2.5-flash",  
        contents=[image, prompt_text],  
        config=types.GenerateContentConfig(  
            system_instruction=system_instruction,  
            response_mime_type="application/json",  
            response_schema=BrandAssetAudit,  
            temperature=0.2,  
        ),  
    )  
  
    return BrandAssetAudit.model_validate_json(response.text)  
  
# --- 3. Streamlit User Interface ---  
st.title("🎨 BrandPulse AI")  
st.subheader("Senior Brand & UX Auditor Dashboard")  
st.markdown("---")  
  
# Setup Sidebar for Inputs  
st.sidebar.header("🔑 Authentication & Context")  
  
# Look for environment variable first, or allow manual input  
env_key = os.environ.get("GEMINI_API_KEY", "")  
api_key = st.sidebar.text_input("Gemini API Key", value=env_key, type="password", help="Get a key from Google AI Studio")  
  
st.sidebar.markdown("---")  
st.sidebar.header("📝 Audit Setup")  
brand_context = st.sidebar.text_area(  
    "Target Audience & Brand Context",  
    value="B2B SaaS homepage targeting engineering leaders, optimizing for signups",  
    placeholder="Describe the goals, target audience, or specific requirements..."  
)  
  
uploaded_file = st.sidebar.file_picker(  
    "Upload Brand Artifact",  
    type=["png", "jpg", "jpeg", "webp"],  
    help="Upload a screenshot of your landing page, mobile UI, or component layout"  
)  
  
# Main Application Window Layout  
if not api_key:  
    st.info("👋 Welcome! Please enter your **Gemini API Key** in the sidebar to get started.")  
elif not uploaded_file:  
    st.info("📸 Upload a website screenshot or brand artifact in the sidebar to run the audit.")  
else:  
    # Split layout: Preview image vs Trigger action  
    col1, col2 = st.columns([1, 2])  
      
    with col1:  
        st.image(uploaded_file, caption="Artifact Pending Review", use_container_width=True)  
        run_audit_btn = st.button("🚀 Run Diagnostic Audit", type="primary", use_container_width=True)  
  
    with col2:  
        if run_audit_btn:  
            with st.spinner("Analyzing visual architecture, calculating contrast scores, and compiling roadmap..."):  
                try:  
                    # Run engine  
                    result = run_brandpulse_audit(uploaded_file, api_key, brand_context)  
                      
                    # Cache result in session state to prevent losing it on page interaction  
                    st.session_state["audit_result"] = result  
                    st.success("Audit Completed Successfully!")  
                except Exception as e:  
                    st.error(f"Error executing audit: {e}")  
  
        # If results exist, build the professional presentation layout  
        if "audit_result" in st.session_state:  
            res = st.session_state["audit_result"]  
              
            # --- ROW 1: Executive Scores ---  
            st.markdown("### 📊 Executive Overview")  
            m1, m2, m3, m4 = st.columns(4)  
            m1.metric("Overall Health", f"{res.overallHealthScore}/100")  
            m2.metric("Typography Pillar", f"{res.pillars.visualHierarchyAndTypography.score}/100")  
            m3.metric("Color Harmony Pillar", f"{res.pillars.colorPaletteAndBrandConsistency.score}/100")  
            m4.metric("UX & WCAG Pillar", f"{res.pillars.uxAndAccessibility.score}/100")  
              
            st.info(f"**Brand Maturity Tier:** {res.brandMaturityLevel}")  
              
            # --- ROW 2: Contextual Analysis ---  
            tab1, tab2, tab3 = st.tabs(["📝 Executive Analysis", "🎨 Design Pillars", "🛠️ Growth Roadmap"])  
              
            with tab1:  
                st.markdown("#### Executive Summary")  
                st.write(res.executiveSummary)  
                st.markdown("#### Business Growth & Conversion Impact")  
                st.write(res.businessGrowthImpact)  
                  
            with tab2:  
                # Pillar 1 Details  
                st.markdown("#### 1. Visual Hierarchy & Typography")  
                st.write(f"**Primary Header Legibility:** {res.pillars.visualHierarchyAndTypography.primaryHeaderLegibility}")  
                st.write(f"**Font Pairing Critique:** {res.pillars.visualHierarchyAndTypography.fontPairingCritique}")  
                st.write(f"**Readability:** {res.pillars.visualHierarchyAndTypography.readability.status} — {res.pillars.visualHierarchyAndTypography.readability.assessment}")  
                st.caption(f"💡 *Recommendation:* {res.pillars.visualHierarchyAndTypography.recommendation}")  
                st.markdown("---")  
                  
                # Pillar 2 Details  
                st.markdown("#### 2. Color Palette & Brand Consistency")  
                st.write(f"**Harmony Paradigm:** {res.pillars.colorPaletteAndBrandConsistency.colorHarmony.paletteType} ({res.pillars.colorPaletteAndBrandConsistency.colorHarmony.assessment})")  
                st.write(f"**WCAG Status:** {res.pillars.colorPaletteAndBrandConsistency.contrastRatios.wcagComplianceStatus}")  
                  
                # Render color swatches dynamically  
                st.write("**Dominant Palette Colors Detected:**")  
                color_cols = st.columns(len(res.pillars.colorPaletteAndBrandConsistency.dominantBrandColors))  
                for idx, hex_c in enumerate(res.pillars.colorPaletteAndBrandConsistency.dominantBrandColors):  
                    with color_cols[idx]:  
                        st.markdown(  
                            f"<div style='background-color:{hex_c}; height:40px; border-radius:5px; border:1px solid #ddd; text-align:center; line-height:40px; color:#fff; font-weight:bold; font-size:12px; text-shadow: 1px 1px 2px #000;'>{hex_c}</div>",   
                            unsafe_allow_html=True  
                        )  
                st.caption(f"💡 *Recommendation:* {res.pillars.colorPaletteAndBrandConsistency.recommendation}")  
                st.markdown("---")  
                  
                # Pillar 3 Details  
                st.markdown("#### 3. UX & Accessibility Metrics")  
                st.write(f"**Accessibility Profile:** {res.pillars.uxAndAccessibility.wcagComplianceTier}")  
                st.write(f"**Call-to-Action (CTA) Salience:** Score {res.pillars.uxAndAccessibility.ctaVisibility.salienceScore}/100 — {res.pillars.uxAndAccessibility.ctaVisibility.critique}")  
                  
                if res.pillars.uxAndAccessibility.lowContrastElements:  
                    st.write("**Identified Low-Contrast Flagged Elements:**")  
                    lc_df = pd.DataFrame([item.model_dump() for item in res.pillars.uxAndAccessibility.lowContrastElements])  
                    st.dataframe(lc_df, use_container_width=True)  
  
  
