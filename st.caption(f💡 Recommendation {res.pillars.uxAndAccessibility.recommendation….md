st.caption(f"💡 *Recommendation:* {res.pillars.uxAndAccessibility.recommendation}")  
with tab3:  
st.markdown("#### Prioritized Growth & Conversion Roadmap")  
roadmap_data = []  
for idx, item in enumerate(res.actionableGrowthRoadmap):  
roadmap_data.append({  
"Priority": item.priority,  
"Action Plan Title": item.title,  
"Impact Area": item.impactArea,  
"Implementation Effort": item.effort,  
"Strategic Advisory": item.recommendation  
})  
df_roadmap = pd.DataFrame(roadmap_data)  
st.dataframe(df_roadmap, use_container_width=True)  
