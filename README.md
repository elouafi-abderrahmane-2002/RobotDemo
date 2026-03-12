# 🤖 Robot Framework Demo — Keyword-Driven, Data-Driven & Gherkin

Robot Framework est un framework d'automatisation générique — mais la façon
dont on structure les tests change tout. Ce projet démontre trois approches
différentes sur la même application cible (une calculatrice Python), chacune
adaptée à un contexte différent.

---

## Les trois approches de test

```
  Application testée : Calculator (Python)
          │
          ├── keyword_driven.robot    ← tests structurés par mots-clés
          ├── data_driven.robot       ← même test, données multiples
          ├── gherkin.robot           ← BDD Given/When/Then
          └── CalculatorLibrary.py    ← bibliothèque de mots-clés custom
```

---

## Approche 1 : Keyword-Driven

Chaque action métier est encapsulée dans un mot-clé réutilisable.

```robotframework
*** Settings ***
Library    CalculatorLibrary

*** Test Cases ***
Addition Should Work
    [Documentation]    Vérifie que l'addition de deux entiers est correcte
    Push Button    1
    Push Button    +
    Push Button    2
    Push Button    =
    Result Should Be    3

Division By Zero Should Fail
    [Documentation]    Vérifie que la division par zéro lève une erreur
    Push Button    6
    Push Button    /
    Push Button    0
    Push Button    =
    Should Be Error    Cannot divide by zero

*** Keywords ***
Result Should Be
    [Arguments]    ${expected}
    ${result}=    Get Result
    Should Be Equal As Numbers    ${result}    ${expected}
```

---

## Approche 2 : Data-Driven

Un seul cas de test exécuté avec de multiples jeux de données — idéal pour
tester des règles métier sur de nombreuses combinaisons.

```robotframework
*** Settings ***
Library           CalculatorLibrary
Test Template     Calculate

*** Test Cases ***       EXPRESSION    EXPECTED
Addition simple          1 + 2         3
Addition négative        -1 + -2       -3
Soustraction             10 - 3        7
Multiplication           4 * 5         20
Division entière         10 / 2        5
Priorité opérateurs      2 + 3 * 4     14
Grand nombre             999 + 1       1000

*** Keywords ***
Calculate
    [Arguments]    ${expression}    ${expected}
    ${result}=    Evaluate Expression    ${expression}
    Should Be Equal As Numbers    ${result}    ${expected}
```

---

## Approche 3 : BDD Gherkin

Collaboration entre équipes techniques et métier — les scénarios sont lisibles
par tous, pas seulement par les développeurs.

```robotframework
*** Settings ***
Library    CalculatorLibrary

*** Test Cases ***
Addition de deux nombres positifs
    Given une calculatrice vide
    When je saisie    2
    And  je saisie    +
    And  je saisie    3
    And  j appuie sur égal
    Then le résultat doit être    5

Division par zéro est rejetée
    Given une calculatrice vide
    When je saisie    5
    And  je saisie    /
    And  je saisie    0
    And  j appuie sur égal
    Then une erreur doit s afficher    Cannot divide by zero

*** Keywords ***
une calculatrice vide
    Reset Calculator

je saisie
    [Arguments]    ${input}
    Push Button    ${input}

j appuie sur égal
    Push Button    =

le résultat doit être
    [Arguments]    ${expected}
    ${result}=    Get Result
    Should Be Equal As Numbers    ${result}    ${expected}

une erreur doit s afficher
    [Arguments]    ${message}
    ${error}=    Get Error
    Should Contain    ${error}    ${message}
```

---

## Bibliothèque custom Python

```python
# CalculatorLibrary.py
from robot.api.deco import keyword

class CalculatorLibrary:
    """Bibliothèque Robot Framework pour tester une calculatrice Python."""

    def __init__(self):
        self._calc = Calculator()

    @keyword("Push Button")
    def push_button(self, button: str):
        """Simule l'appui sur un bouton de la calculatrice."""
        self._calc.push(button)

    @keyword("Get Result")
    def get_result(self) -> float:
        """Retourne le résultat affiché."""
        return self._calc.result

    @keyword("Reset Calculator")
    def reset(self):
        """Remet la calculatrice à zéro."""
        self._calc.reset()
```

---

## Lancer les tests

```bash
git clone https://github.com/elouafi-abderrahmane-2002/robotframework-demo.git
cd robotframework-demo
pip install robotframework

robot keyword_driven.robot
robot data_driven.robot
robot gherkin.robot
robot --outputdir results .         # tous les tests + rapport HTML
```

---

## Ce que j'ai appris

L'approche **data-driven** est souvent sous-estimée. Sur un cas comme la calculatrice,
tester 7 combinaisons avec un seul cas de test (au lieu de 7 tests séparés) divise
par 7 le code à maintenir. Si la logique du test change, on modifie un seul endroit.

Le **BDD/Gherkin** n'est utile que si les scénarios sont vraiment rédigés avec
les parties prenantes métier. Si seuls les développeurs les lisent, keyword-driven
est plus efficace. Le choix dépend de qui valide les tests — pas de la technologie.

---

*Projet réalisé dans le cadre de ma formation ingénieur — ENSET Mohammedia*
*Par **Abderrahmane Elouafi** · [LinkedIn](https://www.linkedin.com/in/abderrahmane-elouafi-43226736b/) · [Portfolio](https://my-first-porfolio-six.vercel.app/)*
