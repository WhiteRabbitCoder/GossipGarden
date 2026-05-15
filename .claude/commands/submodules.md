---
name: submodules
description: >
  Guía para trabajar con los submódulos de git en este metarepo (GossipGarden).
  Usar cuando el usuario pregunte cómo clonar el repo completo, actualizar submódulos,
  hacer push de cambios en frontend o backend, o sincronizar el metarepo con los últimos commits.
---

# Submódulos en GossipGarden

Este metarepo contiene dos submódulos:

| Submódulo | Repositorio |
|---|---|
| `frontendGossipGarden/` | `git@github.com:WhiteRabbitCoder/FrontendGossipGarden.git` |
| `backendGossipGarden/` | `git@github.com:Danieloid3/BackendGossipGarden.git` |

---

## Clonar el metarepo por primera vez

```bash
git clone --recurse-submodules git@github.com:WhiteRabbitCoder/GossipGarden.git
cd GossipGarden
```

Si ya clonaste sin `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

---

## Actualizar submódulos al último commit de cada rama

```bash
# Actualizar todos los submódulos a lo que apunta el metarepo
git submodule update --recursive

# Actualizar cada submódulo al HEAD de su rama remota
git submodule update --remote --merge
```

---

## Hacer cambios en un submódulo y pushearlos

Los submódulos son repos independientes. El flujo es:

```bash
# 1. Entrar al submódulo
cd frontendGossipGarden        # o backendGossipGarden

# 2. Asegurarte de estar en la rama correcta
git checkout main              # o la rama que corresponda

# 3. Hacer los cambios, commits y push normalmente
git add .
git commit -m "feat(scope): descripción"
git push origin main

# 4. Volver al metarepo y registrar el nuevo puntero
cd ..
git add frontendGossipGarden   # el metarepo registra el nuevo commit SHA
git commit -m "chore(submodules): update frontendGossipGarden pointer"
git push origin main
```

> El metarepo no guarda el código de los submódulos, solo un **puntero al commit exacto** de cada uno. Cada vez que avance un submódulo, hay que actualizar ese puntero en el metarepo.

---

## Pull — traer cambios de ambos submódulos

```bash
# Desde la raíz del metarepo
git pull origin main
git submodule update --remote --merge
```

O de un submódulo específico:

```bash
cd backendGossipGarden
git pull origin main
cd ..
git add backendGossipGarden
git commit -m "chore(submodules): update backendGossipGarden pointer"
git push origin main
```

---

## Ver el estado actual de los submódulos

```bash
git submodule status
```

Muestra el commit SHA al que apunta el metarepo para cada submódulo. Un `+` al inicio indica que el submódulo tiene commits locales que aún no se registraron en el metarepo.

---

## Reglas clave

- **Nunca edites archivos de un submódulo desde la raíz del metarepo.** Entra al directorio del submódulo primero.
- **Siempre pushea el submódulo antes de actualizar el puntero en el metarepo.** Si el metarepo apunta a un commit que no existe en el remoto, los demás no podrán clonarlo.
- El metarepo sigue el mismo git-flow que los submódulos: `rama → qa → main`. Aplica `/git-flow` para commits y ramas en el metarepo también.
