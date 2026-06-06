# Spanish-language UI standards

Applies to projects with a Spanish-speaking end-user audience (the default for
most of my work). Engineering communication, code identifiers, and commit
messages remain in **English**; only the user-facing surface is in Spanish.

## What this covers

- Visible strings: labels, buttons, headings, placeholders, helper text.
- System messages: errors, toasts, confirmations, empty states.
- Email and notification copy that reaches an end user.

## What this does NOT cover

- Variable names, function names, file names — always English.
- Code comments — always English.
- Commit messages, PR titles, and PR descriptions — always English.
- Internal logs, debug output, and developer-facing CLI tooling — English.

## Translation conventions

- Use **usted** for formal contexts (B2B SaaS, financial, healthcare).
- Use **tú** for consumer apps, casual products, and chat surfaces.
- Pick one within a project and stay consistent — never mix.

## Common UI vocabulary

| English        | Spanish (formal)     | Spanish (informal)   |
|----------------|----------------------|----------------------|
| Sign in        | Iniciar sesión       | Iniciar sesión       |
| Sign up        | Crear cuenta         | Crear cuenta         |
| Sign out       | Cerrar sesión        | Cerrar sesión        |
| Submit         | Enviar               | Enviar               |
| Save           | Guardar              | Guardar              |
| Cancel         | Cancelar             | Cancelar             |
| Delete         | Eliminar             | Eliminar             |
| Edit           | Editar               | Editar               |
| Search         | Buscar               | Buscar               |
| Loading…       | Cargando…            | Cargando…            |
| Try again      | Vuelva a intentarlo  | Inténtalo de nuevo   |
| Required field | Campo obligatorio    | Campo obligatorio    |

## Pitfalls to avoid

- Don't machine-translate UI strings literally. "Submit" → "Someter" reads
  badly; the correct verb is almost always "Enviar" or "Confirmar".
- Inverted question/exclamation marks (`¿`, `¡`) at the start of clauses are
  not optional in Spanish UI copy.
- Date and number formats: `dd/mm/yyyy` and `,` as thousands separator with `.`
  as decimal in most LATAM contexts. Confirm per-project; CDMX and Buenos Aires
  may differ.
- Currency: prefix with the symbol and a space (`$ 1,250.00 MXN`), and always
  show the currency code on amounts that could be ambiguous (MXN vs USD).
