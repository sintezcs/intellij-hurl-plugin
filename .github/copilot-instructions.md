# IntelliJ Hurl Plugin Development Instructions

**CRITICAL: Always follow these instructions exactly. Only fallback to additional search and context gathering if the information here is incomplete or found to be in error.**

This is an IntelliJ IDEA plugin that adds language support for Hurl (https://hurl.dev/), including syntax highlighting, parsing, run configurations, and language injections for JSON, XML, JSONPath, and XPath.

## Quick Start Commands

**MOST COMMON TASKS**:
```bash
# 1. Generate lexer (ALWAYS WORKS - takes 2-3 seconds)
java -jar jflex-1.9.1.jar src/main/java/com/github/jazzytomato/hurl/language/_HurlLexer.flex

# 2. Build plugin (NETWORK REQUIRED - takes 15-45 minutes)
./gradlew buildPlugin
# NEVER CANCEL - Set timeout to 60+ minutes

# 3. Run plugin in sandbox (NETWORK REQUIRED - takes 10-20 minutes)  
./gradlew runIde
# NEVER CANCEL - Set timeout to 30+ minutes

# 4. Run tests (NETWORK REQUIRED - takes 15-30 minutes)
./gradlew check
# NEVER CANCEL - Set timeout to 45+ minutes
```

## Prerequisites & Setup

### Java Version
- **REQUIRED**: Java 17 or higher
- Check version: `java -version`
- The project uses JVM toolchain 17 as configured in build.gradle.kts

### Development Environment
- IntelliJ IDEA (Community or Ultimate) - recommended for plugin development
- Gradle 8.6 (included via wrapper)

## Build System Overview

This project uses:
- **Gradle** with Kotlin DSL (build.gradle.kts)
- **IntelliJ Platform Plugin** template
- **JFlex 1.9.1** for lexer generation  
- **Grammar-Kit** for BNF parser generation
- **Java + Kotlin** mixed language project

## Known Network Limitations

**CRITICAL NETWORK ISSUE**: The build system requires access to `cache-redirector.jetbrains.com` which may be blocked in some environments.

### When Network Access is LIMITED:
- Full `./gradlew` builds will FAIL with "cache-redirector.jetbrains.com: No address associated with hostname"
- Use manual approaches for lexer generation and code development
- Focus on source code editing and local file generation

### When Network Access is AVAILABLE:
- All Gradle tasks should work normally
- Full plugin building and testing possible

## Core Development Tasks

### 1. Lexer Generation (ALWAYS WORKS)
The project includes JFlex 1.9.1.jar for lexer generation:

```bash
# Generate lexer from Hurl.flex specification
cd /path/to/project
java -jar jflex-1.9.1.jar src/main/java/com/github/jazzytomato/hurl/language/_HurlLexer.flex
```

**TIMING**: Lexer generation takes ~2-3 seconds
**OUTPUT**: Generates `src/main/java/com/github/jazzytomato/hurl/language/_HurlLexer.java`

### 2. Grammar and Parser Files
- **BNF Grammar**: `src/main/java/com/github/jazzytomato/hurl/language/Hurl.bnf`
- **Flex Lexer**: `src/main/java/com/github/jazzytomato/hurl/language/_HurlLexer.flex`
- **Generated Sources**: Located in `src/main/gen/`

### 3. Build Tasks (Network Dependent)

**WHEN NETWORK IS AVAILABLE:**

```bash
# NEVER CANCEL - Build may take 15-45 minutes for first run
./gradlew build
# Timeout: Set to 60+ minutes for safety
```

```bash
# Run tests - NEVER CANCEL - May take 15-30 minutes  
./gradlew test
# Timeout: Set to 45+ minutes for safety
```

```bash
# Build plugin distribution
./gradlew buildPlugin
# Timeout: Set to 30+ minutes
```

**WHEN NETWORK IS LIMITED:**
- Use manual JFlex generation (see above)
- Edit source files directly
- Use IDE-based compilation when possible
- Document network limitations in development notes

### 4. Running the Plugin in Development

**Available Run Configurations** (in `.run/` directory):
- **Run Plugin** (`runIde` task) - Start plugin in sandbox IDE
- **Run Tests** (`check` task) - Run all tests
- **Run Verifications** (`runPluginVerifier` task) - Verify plugin compatibility
- **Run IDE for UI Tests** (`runIdeForUiTests` task) - Start IDE for UI testing
- **Run Qodana** (`qodana` task) - Code quality inspection

**WHEN NETWORK IS AVAILABLE:**
```bash
# Run plugin in sandbox IDE - NEVER CANCEL - Takes 10-20 minutes to start
./gradlew runIde
# Timeout: Set to 30+ minutes for safety
```

```bash
# Run tests (equivalent to "Run Tests" configuration)
./gradlew check
# Timeout: Set to 45+ minutes for safety
```

**Alternative using IntelliJ IDEA:**
- Use run configurations in `.run/` directory
- "Run Plugin" starts sandbox IDE with plugin loaded
- "Run Tests" executes full test suite
- "Run Verifications" runs plugin compatibility checks

### 5. Code Quality and Verification

**WHEN NETWORK IS AVAILABLE:**
```bash
# Run Qodana code inspection - NEVER CANCEL - Takes 20-45 minutes
./gradlew qodana
# Timeout: Set to 60+ minutes
```

```bash
# Run plugin verifier - NEVER CANCEL - Takes 30-60 minutes  
./gradlew runPluginVerifier
# Timeout: Set to 90+ minutes
```

## Project Structure and Navigation

### Key Directories
```
src/main/java/com/github/jazzytomato/hurl/
├── language/          # Core language support
│   ├── Hurl.bnf      # Grammar definition
│   ├── Hurl.flex     # Base lexer specification  
│   ├── _HurlLexer.flex # Generated lexer specification
│   └── psi/          # PSI (Program Structure Interface) classes
├── run/              # Run configuration support
└── resources/
    └── META-INF/
        └── plugin.xml # Plugin configuration

src/main/gen/         # Generated source files (do not edit manually)
```

### Important Files to Know
- **plugin.xml**: Plugin configuration and extension points
- **build.gradle.kts**: Build configuration and dependencies
- **gradle.properties**: Project properties and IntelliJ Platform version
- **sample.hurl**: Sample Hurl file for testing

## Validation and Testing

### Manual Validation Steps

**ALWAYS** perform these validation steps after making changes:

1. **Lexer Generation Test**:
   ```bash
   cd /path/to/intellij-hurl-plugin
   java -jar jflex-1.9.1.jar src/main/java/com/github/jazzytomato/hurl/language/_HurlLexer.flex
   # Expected output: "Writing code to src/main/java/com/github/jazzytomato/hurl/language/_HurlLexer.java"
   # Should complete in 2-3 seconds without errors
   ```

2. **File Structure Validation**:
   ```bash
   # Verify key generated files exist
   ls -la src/main/gen/com/github/jazzytomato/hurl/language/
   # Expected: HurlLexer.java, parser/, psi/ directories
   
   # Verify lexer adapter references generated code correctly
   grep -n "new HurlLexer" src/main/java/com/github/jazzytomato/hurl/language/HurlLexerAdapter.java
   # Expected: Line with "new HurlLexer(null)"
   ```

3. **Sample File Testing**:
   ```bash
   # Verify sample file exists and has content
   wc -l sample.hurl
   # Expected: ~4000 lines of Hurl language examples
   ```
   
   **In IDE validation**:
   - Open `sample.hurl` in the plugin
   - Verify syntax highlighting works for HTTP methods (GET, POST, etc.)
   - Check JSON blocks have proper syntax highlighting
   - Test comment/uncomment functionality (Ctrl+/ or Cmd+/)
   - Create run configuration from gutter icon (should appear next to requests)

4. **Language Injection Testing**:
   ```bash
   # Look for JSON content in sample file
   grep -A 5 -B 5 '{' sample.hurl | head -20
   # Should show JSON blocks that need proper highlighting
   ```
   
   **In IDE validation**:
   - Test JSON highlighting in Hurl request bodies
   - Test XML highlighting in Hurl responses  
   - Test JSONPath expressions (jsonpath "$.field")
   - Test XPath expressions (xpath "//element")

### Full Build Validation (Network Required)

**ONLY when network access is available:**

```bash
# Complete validation sequence - NEVER CANCEL ANY STEP
./gradlew clean                    # 30 seconds
./gradlew build                    # 15-45 minutes  
./gradlew test                     # 15-30 minutes
./gradlew runPluginVerifier        # 30-60 minutes
./gradlew buildPlugin              # 10-20 minutes
```

**CRITICAL**: If any command appears to hang, wait at least 60 minutes before considering alternatives.

## Common Development Patterns

### Adding New Language Features
1. **Always** start by updating the BNF grammar in `Hurl.bnf`
2. Update the flex lexer specification in `_HurlLexer.flex`  
3. Regenerate lexer: `java -jar jflex-1.9.1.jar ...`
4. Update syntax highlighter in `HurlSyntaxHighlighter.java`
5. Test with sample Hurl files

### Adding Run Configuration Features
- Main classes in `src/main/java/com/github/jazzytomato/hurl/run/`
- `HurlRunConfiguration.java` - Core configuration
- `HurlRunConfigurationProducer.java` - Auto-generation from context
- `HurlGutterRunLineMarkerProvider.java` - Gutter icons

### Language Injections
- Files: `JsonToHurlInjector.java`, `XmlToHurlInjector.java`, etc.
- Enable syntax highlighting within Hurl files for embedded languages
- Test thoroughly with complex nested expressions

## Common Command Reference

The following are outputs from frequently used commands. Reference them instead of running bash commands to save time.

### Repository Root Structure
```
ls -la /path/to/intellij-hurl-plugin
.
..
.git/
.github/              # GitHub Actions workflows
.gitignore
.idea/                # IntelliJ IDEA project files  
.run/                 # Run configurations
CHANGELOG.md
LICENSE
README.md
build.gradle.kts      # Main build configuration
gradle/               # Gradle wrapper
gradle.properties     # Project properties
gradlew*              # Gradle wrapper scripts
idea-flex.skeleton    # JFlex skeleton for IntelliJ
jflex-1.9.1.jar      # JFlex lexer generator
qodana.yml           # Code quality configuration
sample.hurl          # Example Hurl file (~4000 lines)
settings.gradle.kts   # Gradle settings
src/                 # Source code
```

### Source Directory Structure
```
find src -type d
src
src/main
src/main/gen          # Generated sources (do not edit manually)
src/main/java         # Java source files
src/main/resources    # Resources (plugin.xml, icons, etc.)
```

### Key Configuration Files

**gradle.properties**:
```
pluginGroup = com.github.jazzytomato.hurl
pluginName = intellij-hurl-plugin
pluginVersion = 0.0.3
platformType = IC
platformVersion = 2024.1.2
platformPlugins = org.intellij.intelliLang, com.intellij.jsonpath:241.14494.150, XPathView:241.15989.65
```

**JFlex Version**:
```
java -jar jflex-1.9.1.jar --version
This is JFlex 1.9.1
```

**Sample File Info**:
```
wc -l sample.hurl
3982 sample.hurl
```

## Troubleshooting

### Network Issues
- **Problem**: `cache-redirector.jetbrains.com: No address associated with hostname`
- **Solution**: Use manual lexer generation and work with existing generated files
- **Document**: Always note network limitations in commit messages

### Generated File Issues  
- **Never edit files in `src/main/gen/` manually**
- Always regenerate using JFlex or Grammar-Kit
- Check timestamps to ensure generated files are current

### Plugin Runtime Issues
- Check `build/idea-sandbox/system/log/idea.log` for errors
- Use "Run Plugin" configuration for debugging
- Verify `plugin.xml` extensions are properly configured

## Time Expectations

| Task | Expected Time | Timeout Setting |
|------|---------------|-----------------|
| JFlex generation | 2-3 seconds | 30 seconds |
| Clean build | 15-45 minutes | 60+ minutes |
| Test suite | 15-30 minutes | 45+ minutes |
| Plugin verification | 30-60 minutes | 90+ minutes |
| IDE startup (runIde) | 10-20 minutes | 30+ minutes |
| Qodana inspection | 20-45 minutes | 60+ minutes |

**NEVER CANCEL** any long-running Gradle task. IntelliJ plugin builds are inherently time-consuming.

### Offline Development Workflow (Limited Network)

When full Gradle builds fail due to network issues:

1. **Focus on Source Code Development**:
   ```bash
   # Generate lexer manually
   java -jar jflex-1.9.1.jar src/main/java/com/github/jazzytomato/hurl/language/_HurlLexer.flex
   
   # Edit core source files
   src/main/java/com/github/jazzytomato/hurl/language/
   src/main/java/com/github/jazzytomato/hurl/run/
   ```

2. **Use IntelliJ IDEA for Development**:
   - Open project in IntelliJ IDEA
   - IDE can compile sources without full Gradle resolution
   - Use "Build Project" for incremental compilation
   - Test lexer changes by recompiling HurlLexerAdapter

3. **Manual Testing Approaches**:
   - Create test Hurl files manually
   - Use sample.hurl for testing language features
   - Verify generated lexer code compiles correctly

## CI/CD Integration

The GitHub Actions workflow (`.github/workflows/build.yml`) includes:
- Gradle wrapper validation
- Build and test execution  
- Qodana code inspection
- Plugin verification across multiple IntelliJ versions
- Artifact generation

**Always ensure** your changes pass the complete CI pipeline before merging.