# test
import streamlit as st
import random

# Configuration de la page
st.set_page_config(page_title="Générateur de Tests", page_icon="📝", layout="centered")

# Initialisation de la base de données temporaire en mémoire (session_state)
if 'questions' not in st.session_state:
    st.session_state['questions'] = []

st.title("📝 Plateforme d'Évaluation Interactive")
st.write("Créez vos questions, puis testez vos connaissances !")

# Création de deux onglets de navigation
tab1, tab2 = st.tabs(["⚙️ Création du Test", "🎓 Passer le Test"])

# ---------------------------------------------------------
# ONGLET 1 : CRÉATION DU TEST
# ---------------------------------------------------------
with tab1:
    st.header("Ajouter une nouvelle question")
    
    with st.form("formulaire_creation", clear_on_submit=True):
        matiere = st.text_input("Matière (ex: Mathématiques, Histoire...)")
        question = st.text_area("Posez votre question *")
        
        st.write("Définissez les options :")
        col1, col2 = st.columns(2)
        with col1:
            bonne_reponse = st.text_input("La BONNE réponse *")
            mauvaise_1 = st.text_input("Mauvaise réponse 1 *")
        with col2:
            mauvaise_2 = st.text_input("Mauvaise réponse 2 *")
            mauvaise_3 = st.text_input("Mauvaise réponse 3")

        soumettre = st.form_submit_button("➕ Ajouter au test")

        if soumettre:
            if question and bonne_reponse and mauvaise_1 and mauvaise_2:
                # On regroupe les options et on les mélange pour que la bonne réponse ne soit pas toujours la première
                options = [bonne_reponse, mauvaise_1, mauvaise_2]
                if mauvaise_3:
                    options.append(mauvaise_3)
                random.shuffle(options)

                # Sauvegarde de la question
                st.session_state['questions'].append({
                    "matiere": matiere,
                    "question": question,
                    "options": options,
                    "reponse_correcte": bonne_reponse
                })
                st.success("✅ Question ajoutée avec succès !")
            else:
                st.error("⚠️ Veuillez remplir les champs obligatoires (*).")

    # Affichage du nombre de questions
    if st.session_state['questions']:
        st.info(f"📊 Votre test contient actuellement {len(st.session_state['questions'])} question(s).")

# ---------------------------------------------------------
# ONGLET 2 : PASSER LE TEST
# ---------------------------------------------------------
with tab2:
    st.header("C'est l'heure du test !")
    
    if not st.session_state['questions']:
        st.warning("Aucune question n'est disponible pour le moment. Allez dans l'onglet 'Création du Test'.")
    else:
        score = 0
        reponses_utilisateur = {}
        
        # Affichage des questions
        with st.form("formulaire_test"):
            for i, q in enumerate(st.session_state['questions']):
                st.subheader(f"Question {i+1} : {q['matiere']}")
                # Utilisation d'un bouton radio pour choisir la réponse
                choix = st.radio(q['question'], q['options'], key=f"q_{i}", index=None)
                reponses_utilisateur[i] = choix
            
            validation_test = st.form_submit_button("✔️ Valider mes réponses")
            
            # Calcul du score à la soumission
            if validation_test:
                for i, q in enumerate(st.session_state['questions']):
                    if reponses_utilisateur[i] == q['reponse_correcte']:
                        score += 1
                
                # Affichage du résultat final
                st.divider()
                st.success(f"🎉 TEST TERMINÉ ! Votre score final est de : **{score} / {len(st.session_state['questions'])}**")
                
                # Barre de progression du score
                pourcentage = int((score / len(st.session_state['questions'])) * 100)
                st.progress(pourcentage, text=f"Taux de réussite : {pourcentage}%")
