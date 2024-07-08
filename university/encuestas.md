# Script para rellenar encuestas automaticamente SIU GUARANÍ


### Disclaimer: Recomiendo completar las encuestas honestamente

##### Pero hay veces que uno no fue nunca a clases y aprobó de todas formas, etc.

- podes cambiarle el `value="0"` por inner message `"1"` o algo asi para que te agarre el peor puntaje

```javascript

// Get all the radio buttons
const radioButtons = document.querySelectorAll('input[type="radio", value="0"]');

// Iterate through each radio button
radioButtons.forEach(radioButton => {
    radioButton.checked = true
});
```