# Cursos - Educational Content Repository

**ALWAYS follow these instructions first and fallback to search or bash commands only when encountering unexpected information that does not match the info here.**

This repository contains Spanish-language educational programming courses organized as modular documentation. This is a **documentation-only repository** with no source code, build systems, or executable applications.

## Repository Structure

The repository contains 86 markdown files across 6 main course directories plus a base template:

```
cursos/
├── README.md (main repository overview)
├── LICENSE.md (MIT license)
├── .gitignore
├── Curso-Base/ (9 files - course template)
├── blockchain-ethereum-solidity/ (23 files - blockchain development)
├── curso_C#/ (9 files - C# and .NET programming)
├── curso_gestion_despliegue_proyectos/ (13 files - project management)
├── estrategia-testing-profesional/ (9 files - testing strategies)
├── git_github/ (11 files - version control)
└── java/ (9 files - Java programming)
```

## Working Effectively

### CRITICAL: This is Documentation Only
- **DO NOT attempt to build, compile, or run code** - there are no source code files
- **DO NOT look for package.json, *.csproj, or build scripts** - none exist
- **DO NOT try to install dependencies or run tests** - this is pure documentation
- **Focus on content validation, markdown structure, and educational material organization**

### Navigate the Repository
- Start with the main `README.md` to understand course offerings
- Each course directory contains:
  - A descriptive `README.md` with course overview, objectives, and structure
  - Numbered modules (e.g., `01-introduccion.md`, `02-conceptos.md`)
  - Individual `LICENSE.md` files for course-specific terms
- Use `find . -name "*.md" | head -20` to explore course content
- Use `ls -la [course-directory]/` to see module organization

### Validate Content Structure
```bash
# Count total markdown files (should be ~86)
find . -name "*.md" | wc -l

# Check each course has proper module structure
for dir in */; do 
  echo "=== $dir ==="
  ls "$dir" | grep "\.md$" | sort
done

# Verify README files exist in each course (should find 8 total)
find . -maxdepth 2 -name "README.md"

# Check for empty content files that need content (currently ~12)
find . -name "*.md" -size 0
```

### Content Standards and Validation
- **Module naming**: Use format `NN-descriptive-name.md` (e.g., `01-introduccion.md`)
- **Course structure**: Each course should have README.md + numbered modules + LICENSE.md
- **Language**: All content is in Spanish
- **Encoding**: UTF-8 text files
- **Line endings**: Unix-style (LF)

### Common Tasks

#### View Course Overview
```bash
# List all courses
ls -d */ | grep -v ".git"

# Read main repository description
cat README.md

# View specific course details
cat [course-directory]/README.md
```

#### Content Navigation
```bash
# Find specific topics across all courses
grep -r "keyword" . --include="*.md"

# List modules in a specific course
ls [course-directory]/*.md | sort

# Check course completeness (modules 01-07 typical)
ls blockchain-ethereum-solidity/ | grep -E "^[0-9]+-"
```

#### Quality Checks
```bash
# Find files with encoding issues
file -i **/*.md | grep -v "utf-8"

# Check for broken internal references (manual review)
grep -r "\]\(" . --include="*.md" | head -10

# Verify all courses have required files
for dir in */; do
  if [[ -d "$dir" && "$dir" != ".git/" ]]; then
    echo "=== $dir ==="
    ls "$dir" | grep -E "(README|LICENSE)\.md"
  fi
done
```

## Course-Specific Information

### Blockchain Ethereum Solidity (23 modules)
- Comprehensive blockchain development course
- Covers Solidity, smart contracts, and dApps
- Advanced topics: security, gas optimization, design patterns
- Key files: `03-fundamentos-solidity.md`, `13-seguridad-avanzada-smart-contracts.md`

### C# Course (9 modules)
- Modern C# and .NET 8 development
- Object-oriented programming, web development with ASP.NET Core
- Enterprise architecture patterns
- Key files: `02-poo.md`, `04-web.md`

### Java Course (9 modules)
- Basic to advanced Java programming
- OOP concepts, collections, enterprise applications
- JDK 17+ focused
- Key files: `03-setup.md`, `04-practica.md`

### Git & GitHub (11 modules)
- Version control and collaboration
- GitHub Actions and automation
- Professional workflows
- Key files: `06-automatizacion.md`, `07-colaboracion.md`

### Project Management (13 modules)
- Software project lifecycle
- Agile methodologies, CI/CD, containers
- Infrastructure as code, cloud deployment
- Key files: `06-cicd.md`, `07-contenedores.md`

### Testing Strategy (9 modules)
- Professional testing approaches
- Tools: Jest, Cypress, Playwright
- CI/CD integration
- Key files: `03-setup.md`, `05-integracion.md`

## Content Creation and Maintenance

### Adding New Content
1. **Follow the established pattern**: README.md + numbered modules + LICENSE.md
2. **Use consistent naming**: `NN-descriptive-name.md` format
3. **Include proper headers**: Each module should start with `# NN. Module Title`
4. **Add to course README**: Update module list in the course README.md

### Updating Existing Content
1. **Maintain module numbering**: Don't renumber existing modules
2. **Check cross-references**: Ensure links between modules remain valid
3. **Update course README**: Reflect any structural changes
4. **Preserve Spanish language**: All content should remain in Spanish

### Validation Workflow
```bash
# 1. Check file structure
find . -name "*.md" | grep -E "^[0-9]+" | sort

# 2. Verify encoding and formatting
file -i **/*.md | grep -v "charset=utf-8" || echo "All files UTF-8"

# 3. Check for empty files that should have content
find . -name "*.md" -size 0 | grep -v "LICENSE.md"

# 4. Validate basic markdown structure
grep -c "^# " **/*.md | grep ":0$" || echo "All files have headers"
```

## Repository Maintenance

### Regular Checks
- **Monthly**: Review for empty content files and complete them
- **Quarterly**: Update course objectives and technology versions
- **As needed**: Add new courses following established patterns

### Common Issues and Solutions
- **Empty module files**: Several courses have placeholder modules (size 0) - these need content
- **Encoding issues**: Ensure all files remain UTF-8
- **Inconsistent structure**: Some courses may deviate from standard numbering
- **Missing cross-references**: Modules should link to related content where appropriate

### Performance Notes
- **Repository size**: ~85 markdown files, minimal storage impact
- **Navigation time**: Instant - all text files
- **Search operations**: Very fast with grep across markdown files
- **No build time**: Pure documentation requires no compilation

## Troubleshooting

### Common Scenarios
**"I can't find build instructions"**: This is documentation-only - no builds needed
**"Tests are failing"**: No test suite exists - this is educational content
**"Application won't start"**: No executable application - only markdown documentation
**"Dependencies missing"**: No dependencies - pure text content

### Content Issues
```bash
# Find courses with incomplete modules
find . -name "*.md" -size 0 | sort

# Check for courses missing standard structure
for dir in */; do
  if [[ -d "$dir" && "$dir" != ".git/" ]]; then
    count=$(ls "$dir"/*.md 2>/dev/null | wc -l)
    echo "$dir: $count modules"
  fi
done
```

### Quick Repository Health Check
```bash
# Verify structure integrity  
echo "Total courses: $(ls -d */ | grep -v ".git" | wc -l)"  # Should be 6
echo "Total markdown files: $(find . -name "*.md" | wc -l)"  # Should be 86
echo "Empty files needing content: $(find . -name "*.md" -size 0 | wc -l)"  # Currently 12
echo "Courses with README: $(find . -maxdepth 2 -name "README.md" | wc -l)"  # Should be 8
```

**Remember**: This repository is about educational content management, not software development. Focus on documentation quality, course structure, and learning material organization rather than traditional development workflows.