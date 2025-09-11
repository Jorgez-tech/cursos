# 07. Colaboración Avanzada y Gestión de Proyectos

## 🤝 Colaboración Efectiva con GitHub

GitHub va más allá del control de versiones: es una plataforma completa de colaboración que facilita la gestión de proyectos, comunicación entre equipos y organización de tareas.

### Filosofía de colaboración abierta
- **Transparencia**: Todo el trabajo es visible para el equipo
- **Asincronía**: Colaboradores en diferentes zonas horarias
- **Documentación**: Decisiones y discusiones quedan registradas
- **Calidad**: Revisiones de código y procesos establecidos

## 📝 Issues: Centro de Comunicación

### ¿Qué son los Issues?
Los issues son el sistema de tickets de GitHub para:
- **Bug reports**: Reportar errores y problemas
- **Feature requests**: Solicitar nuevas funcionalidades  
- **Tasks**: Organizar tareas y recordatorios
- **Preguntas**: Buscar ayuda del equipo
- **Documentación**: Discusiones técnicas

### Crear Issues efectivos

#### Estructura de un buen Issue
```markdown
## 🐛 Descripción del Bug
El botón de envío en el formulario de contacto no responde cuando...

## 📋 Pasos para Reproducir
1. Ir a /contacto
2. Llenar formulario
3. Hacer clic en "Enviar"
4. Observar que no sucede nada

## 🎯 Resultado Esperado
El formulario debería enviarse y mostrar mensaje de confirmación

## 💥 Resultado Actual
Nada sucede, no hay feedback visual

## 🖥️ Información Adicional
- Browser: Chrome 118
- OS: Windows 11  
- Versión: v2.1.0

## 📎 Screenshots
[Adjuntar imagen del problema]
```

### Gestión avanzada de Issues
```bash
# Referenciar issues en commits
git commit -m "Fix login validation, closes #123"

# Keywords que cierran issues automáticamente:
# closes, fixes, resolves, close, fix, resolve
git commit -m "Resolves #45 - Add user authentication"
```

### Labels y organización
- **bug**: Errores que necesitan corrección
- **enhancement**: Mejoras y nuevas características
- **documentation**: Cambios en documentación
- **good first issue**: Ideal para nuevos colaboradores
- **help wanted**: Necesita ayuda de la comunidad
- **priority/high**: Urgente
- **status/in-progress**: En desarrollo

## 📊 GitHub Projects: Gestión Ágil

### Projects v2 (Beta) - Recomendado
```markdown
## Configuración de Proyecto
1. Repository → Projects → New project
2. Seleccionar template (Team backlog, Feature planning)
3. Configurar vistas: Board, Table, Roadmap
4. Customizar campos: Priority, Status, Assignee, Size
```

### Flujo de trabajo con Projects
1. **Backlog**: Ideas y tareas sin empezar
2. **Ready**: Tareas definidas, listas para desarrollo  
3. **In Progress**: En desarrollo activo
4. **In Review**: Pull requests en revisión
5. **Done**: Completadas y deployadas

### Automatizaciones útiles
- **Auto-add issues**: Nuevos issues van a "Backlog"
- **Auto-move PRs**: Pull requests van a "In Review"
- **Auto-close**: Issues se mueven a "Done" al cerrar
- **Status sync**: Sincronizar con labels de issues

### Views personalizadas
```markdown
## Vista de Sprint (Board)
- Filtro: milestone:"Sprint 23" 
- Agrupado por: Status
- Ordenado por: Priority

## Vista de Roadmap 
- Filtro: label:epic
- Timeline por: Target date
- Agrupado por: Team

## Vista de Bugs (Table)
- Filtro: label:bug AND is:open
- Columns: Title, Assignee, Priority, Created
```

## 👥 Organización de Equipos

### Estructura de equipos en GitHub
```markdown
## Organización: MiEmpresa
### Teams:
- **Backend Team**: API y servicios
- **Frontend Team**: UI/UX y cliente  
- **DevOps Team**: Infrastructure y CI/CD
- **QA Team**: Testing y quality assurance

### Permisos por team:
- **Admin**: Full access
- **Write**: Push + PR review
- **Triage**: Issue management
- **Read**: View only
```

### Mejores prácticas para equipos
- **Code owners**: Define responsables por área de código
- **Branch protection**: Requiere reviews antes de merge
- **Team mentions**: @backend-team para notificaciones
- **Team discussions**: Espacio privado para coordinación

### CODEOWNERS file
```text
# Global owners
* @admin-team

# Frontend
/src/frontend/ @frontend-team @ui-lead

# Backend API  
/src/api/ @backend-team @arch-lead

# Infrastructure
/infra/ @devops-team
/docker/ @devops-team
/.github/ @devops-team

# Documentation
/docs/ @docs-team @product-owner
*.md @docs-team
```

## 💬 GitHub Discussions

### Cuándo usar Discussions vs Issues
| Discussions | Issues |
|-------------|--------|
| Preguntas abiertas | Tareas específicas |
| Brainstorming | Bug reports |
| Anuncios | Feature requests |
| Tutoriales | Seguimiento de trabajo |
| Community building | Project management |

### Categorías de Discussions
- **🎤 Announcements**: Noticias y actualizaciones
- **💡 Ideas**: Brainstorming de nuevas características
- **🙋 Q&A**: Preguntas y respuestas
- **🗣️ General**: Conversación general
- **📢 Show and tell**: Mostrar trabajo y proyectos

### Moderar discussions efectivamente
```markdown
## Templates para discussions
### 💡 Idea Template
**Problem**: ¿Qué problema resuelve esta idea?
**Solution**: ¿Cómo la resolverías?  
**Impact**: ¿A quién beneficia?
**Alternative**: ¿Consideraste otras opciones?
```

## 📈 Milestones y Planning

### Crear milestones efectivos
```markdown
## Milestone: v2.0 Release
**Due date**: 2024-03-15
**Description**: Major feature release with user authentication

### Success criteria:
- [ ] User registration/login (5 issues)
- [ ] Dashboard UI redesign (3 issues)  
- [ ] API security improvements (4 issues)
- [ ] Performance optimizations (2 issues)

**Progress**: 8/14 issues completed (57%)
```

### Sprint planning con GitHub
1. **Create milestone** para sprint
2. **Estimate issues** con labels (S/M/L/XL)
3. **Assign issues** a milestone y developers
4. **Track progress** en project board
5. **Review completion** al final del sprint

## 🔄 Advanced Pull Request Workflows

### PR Templates
```markdown
<!-- .github/pull_request_template.md -->
## 📝 Summary
Brief description of changes

## 🏗️ Type of Change
- [ ] Bug fix
- [ ] New feature  
- [ ] Breaking change
- [ ] Documentation update

## 🧪 Testing
- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Manual testing completed

## 📋 Checklist
- [ ] Self-review completed
- [ ] Code follows style guidelines  
- [ ] Documentation updated
- [ ] Ready for review

## 📎 Related Issues
Closes #123, Related to #456
```

### Review process optimization
```markdown
## Review Guidelines
### For Authors:
- Keep PRs small (< 400 lines)
- Self-review before requesting
- Provide context in description
- Respond to feedback promptly

### For Reviewers:  
- Review within 24 hours
- Be constructive and specific
- Focus on logic, not style
- Approve when satisfied
```

### Automated checks integration
```yaml
# .github/workflows/pr-checks.yml
name: PR Checks
on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: npm test
      
      - name: Check coverage
        run: npm run coverage
        
      - name: Lint code
        run: npm run lint
```

## 🏋️ Ejercicios Prácticos

### Ejercicio 1: Issues workflow completo
1. Crea issue de bug con template completo
2. Asigna labels apropiados
3. Crea branch desde issue
4. Soluciona bug y referencía issue en commit
5. Verifica que issue se cierre automáticamente

### Ejercicio 2: Project board ágil
1. Crea proyecto con template "Team backlog"
2. Agrega 10 issues de ejemplo
3. Configura automatizaciones
4. Simula sprint planning moviendo issues
5. Crea vistas personalizadas (board, roadmap, table)

### Ejercicio 3: Team collaboration
1. Invita 2 colaboradores a repositorio
2. Asigna diferentes roles y permisos
3. Crea CODEOWNERS file
4. Configura branch protection con required reviews
5. Practica code review workflow

### Ejercicio 4: Release planning
1. Crea milestone para próximo release
2. Agrupa issues relacionados
3. Estima esfuerzo con labels
4. Asigna issues a milestone
5. Simula sprint review con progreso

## 🎯 Mejores Prácticas de Colaboración

### Comunicación efectiva
- **Be specific**: Detalles claros en issues y PRs
- **Be kind**: Tono constructivo en reviews
- **Be responsive**: Responde feedback oportunamente
- **Be transparent**: Comparte progreso y blockers

### Organización de trabajo
- **Small PRs**: Más fáciles de revisar y mergear
- **Clear naming**: Branches, PRs e issues descriptivos
- **Regular updates**: Status en issues y PRs
- **Documentation**: Contexto para decisiones

### Gestión de comunidad
- **Welcome newcomers**: Good first issues y mentoring
- **Recognition**: Agradece contribuciones
- **Guidelines**: Contribución y código de conducta claros
- **Consistency**: Procesos predecibles y documentados

## 🔧 Herramientas de Productividad

### GitHub CLI para colaboración
```bash
# Crear issue desde terminal
gh issue create --title "Fix login bug" --body "Users cannot log in"

# Asignar issue
gh issue edit 123 --assignee @me

# Ver issues asignados
gh issue list --assignee @me

# Crear PR desde branch
gh pr create --title "Fix authentication" --body "Resolves login issues"

# Review PR
gh pr review 456 --approve --body "LGTM!"

# Merge PR
gh pr merge 456 --squash
```

### Extensiones útiles
- **GitHub Desktop**: Cliente gráfico oficial
- **GitLens (VS Code)**: Blame inline y historial
- **GitHub PR (VS Code)**: Reviews desde editor
- **Refined GitHub**: Mejoras UX para browser
- **Octotree**: Tree view de repositorios

---
**Anterior**: [06. Automatización y DevOps con GitHub Actions](./06-automatizacion.md) | **Siguiente**: [08. Mantenimiento, Auditoría y Eficiencia](./08-mantenimiento.md)