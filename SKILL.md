---
name: ibm-odm
description: >
  Use this skill whenever the user works with IBM Operational Decision Manager (ODM) projects — 
  versions 8.9, 8.10, 8.11, or later. Trigger for ANY mention of: ODM, BAL rules, business rules, 
  decision tables, ruleflows (.rfl), XOM, BOM, Rule Execution Server (RES), Decision Center, 
  Decision Server, rule projects, rule packages, decision services, decision operations, 
  IRL files, or Java model classes used as execution/business object models. Also trigger when 
  the user asks to read, create, modify, debug, or understand rule project structures, 
  BAL syntax, decision table conditions/actions, ruleflow diagrams, XOM/BOM mappings, 
  deployment configurations, simulation data providers, or RuleApp archives. 
  Even if the user doesn't say "ODM" explicitly but mentions business rules in a Java context 
  with terms like "ruleset", "rule engine", "BRMS", or "business rule management", use this skill.
---

# IBM ODM Skill

This skill teaches Claude how to read, understand, create, and modify artifacts in IBM Operational Decision Manager (ODM) projects. ODM is IBM's Business Rule Management System (BRMS) used extensively in financial services, insurance, healthcare, and government for automating complex business decisions.

## Why this matters

ODM projects have a very specific file structure and XML-based formats that look deceptively simple but have strict schemas and interdependencies. Getting a single namespace wrong, omitting a required UUID, or misunderstanding the BOM-to-XOM mapping will produce artifacts that compile but fail at runtime. This skill provides the precise knowledge needed to work with these files correctly.

---

## ODM Project Structure

A typical ODM rule project has this layout:

```
my-rule-project/
├── .syncproject                    # Decision Center sync metadata
├── deployment/                     # Deployment descriptors
│   └── MyRuleApp.dep              # RuleApp deployment config (XML)
├── bom/
│   └── model_bom.bom             # Business Object Model definition (XML)
├── rules/
│   ├── mypackage/
│   │   ├── rule1.brl             # BAL rule file (XML wrapping BAL text)
│   │   ├── rule2.dta             # Decision table (XML)
│   │   └── rule3.trl             # Technical rule (IRL — ILOG Rule Language)
│   └── anotherpackage/
│       └── ...
├── resources/
│   ├── mypackage/
│   │   └── ruleflow1.rfl         # Ruleflow definition (XML)
│   └── ...
├── templates/                      # Rule templates (optional)
└── queries/                        # Saved queries (optional)
```

Key conventions:
- **Package structure mirrors directories** under `rules/` and `resources/`
- Rule files use the package as their namespace
- The BOM lives in `bom/` and maps Java classes (XOM) to business-friendly names
- Ruleflows orchestrate which rule packages execute and in what order

---

## Core Concepts

### XOM (Execution Object Model)
The XOM is the Java data model that rules execute against at runtime. It consists of:
- **Java classes** (POJOs, beans) compiled into JARs
- Each class has typed fields, getters/setters, and optionally methods
- The XOM JAR is referenced by the rule project but lives outside it (typically in a separate Maven/Gradle project)

When reading an ODM project, look for XOM references in the `.bom` file — each BOM entry maps to a concrete Java class.

### BOM (Business Object Model)
The BOM is a business-friendly abstraction layer over the XOM. It translates Java class names and methods into natural language that business analysts can read.

**BOM file structure** (`.bom` — XML format):
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ilog.rules.studio.model.bom:BOM
    xmlns:ilog.rules.studio.model.bom="http://ilog.rules.studio.model.bom"
    name="myBOM">
  
  <model>
    <!-- Each entry maps a Java class to a business name -->
    <entry>
      <string>com.example.model.Customer</string>
      <value>
        <de.name>Customer</de.name>
        <de.verbalization>a customer</de.verbalization>
        <members>
          <entry>
            <string>age</string>
            <value>
              <de.name>age</de.name>
              <de.verbalization>the age of {this}</de.verbalization>
              <type>int</type>
            </value>
          </entry>
          <entry>
            <string>creditScore</string>
            <value>
              <de.name>credit score</de.name>
              <de.verbalization>the credit score of {this}</de.verbalization>
              <type>int</type>
            </value>
          </entry>
        </members>
      </value>
    </entry>
  </model>
</ilog.rules.studio.model.bom:BOM>
```

Critical BOM details:
- `de.verbalization` defines how the attribute appears in BAL rules — this is what business users see
- `{this}` is a placeholder that gets replaced by the object reference in BAL
- The `<string>` key must exactly match the Java field name in the XOM
- Types map to Java types: `int`, `double`, `java.lang.String`, `java.util.Date`, `boolean`, custom classes

### BAL (Business Action Language)
BAL is the natural-language rule syntax that business analysts write. Each `.brl` file contains XML wrapping a BAL text block.

**BAL rule file structure** (`.brl`):
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ilog.rules.studio.model.rule:ActionRule
    xmlns:ilog.rules.studio.model.rule="http://ilog.rules.studio.model.rule"
    name="CheckCreditEligibility">
  <body language="bal">
    <![CDATA[
definitions
  set 'the customer' to a customer in the customers of the application;
if
  the age of 'the customer' is at least 18
  and the credit score of 'the customer' is at least 650
then
  set the eligibility of 'the customer' to true;
  add "Approved" to the messages of the response;
]]>
  </body>
  <locale>en_US</locale>
  <uuid>5a2f8c91-3b4e-4d7a-9c8f-1234567890ab</uuid>
  <categories>eligibility/credit</categories>
  <priority>0</priority>
</ilog.rules.studio.model.rule:ActionRule>
```

BAL syntax rules:
- `definitions` block declares local variables bound to BOM objects
- `if` block contains conditions using BOM verbalizations
- `then` block contains actions
- Variables are quoted with single quotes: `'the customer'`
- Conditions chain with `and` / `or`
- Comparison operators: `is`, `is not`, `is at least`, `is at most`, `is less than`, `is more than`
- Collection operations: `add X to`, `remove X from`, `there is ... in`, `for each ... in`
- String matching: `matches`, `contains`, `starts with`, `does not contain`

### Decision Tables (`.dta`)

Decision tables encode rule logic as rows and columns. The XML is complex but follows a pattern:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ilog.rules.studio.model.dt:DecisionTable
    xmlns:ilog.rules.studio.model.dt="http://ilog.rules.studio.model.dt"
    name="RiskClassification">
  <body>
    <Structure>
      <ConditionDefinitions>
        <ConditionDefinition>
          <ExpressionDefinition>
            <Text><![CDATA[the credit score of 'the customer']]></Text>
          </ExpressionDefinition>
        </ConditionDefinition>
        <ConditionDefinition>
          <ExpressionDefinition>
            <Text><![CDATA[the income of 'the customer']]></Text>
          </ExpressionDefinition>
        </ConditionDefinition>
      </ConditionDefinitions>
      <ActionDefinitions>
        <ActionDefinition>
          <ExpressionDefinition>
            <Text><![CDATA[set the risk level of 'the customer' to <a string>]]></Text>
          </ExpressionDefinition>
        </ActionDefinition>
      </ActionDefinitions>
    </Structure>
    <Contents>
      <Partition>
        <Condition>
          <Expression>
            <Param><![CDATA[[300..579]]]></Param>
          </Expression>
          <ActionSet>
            <Action>
              <Expression><Param><![CDATA["High"]]></Param></Expression>
            </Action>
          </ActionSet>
        </Condition>
        <Condition>
          <Expression>
            <Param><![CDATA[[580..669]]]></Param>
          </Expression>
          <ActionSet>
            <Action>
              <Expression><Param><![CDATA["Medium"]]></Param></Expression>
            </Action>
          </ActionSet>
        </Condition>
        <Condition>
          <Expression>
            <Param><![CDATA[[670..850]]]></Param>
          </Expression>
          <ActionSet>
            <Action>
              <Expression><Param><![CDATA["Low"]]></Param></Expression>
            </Action>
          </ActionSet>
        </Condition>
      </Partition>
    </Contents>
  </body>
  <locale>en_US</locale>
  <uuid>7b3c9d12-4e5f-6789-abcd-ef0123456789</uuid>
</ilog.rules.studio.model.dt:DecisionTable>
```

Decision table conventions:
- Range syntax: `[min..max]` for inclusive, `(min..max)` for exclusive, combinations allowed
- `<a string>`, `<a number>`, `<a date>` are placeholders in action definitions
- Conditions form a partition tree (nested `Partition` elements for multi-column tables)
- Each row must cover its condition space — gaps cause warnings, overlaps cause conflicts

### Ruleflows (`.rfl`)

Ruleflows define the execution order of rule tasks. They are XML files with a visual graph model:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ilog.rules.studio.model.ruleflow:RuleFlow
    xmlns:ilog.rules.studio.model.ruleflow="http://ilog.rules.studio.model.ruleflow"
    name="MainFlow">
  <body>
    <TaskList>
      <StartTask identifier="start"/>
      <RuleTask identifier="validateInput" 
                packages="validation"
                ordering="literal"
                algorithm="sequential"/>
      <RuleTask identifier="calculateRisk" 
                packages="risk/calculation"
                ordering="literal"
                algorithm="sequential"/>
      <RuleTask identifier="determineAction" 
                packages="actions"
                ordering="literal"
                algorithm="fastpath"/>
      <StopTask identifier="stop"/>
    </TaskList>
    <NodeList>
      <TaskNode identifier="start" task="start"/>
      <TaskNode identifier="validateInput" task="validateInput"/>
      <TaskNode identifier="calculateRisk" task="calculateRisk"/>
      <TaskNode identifier="determineAction" task="determineAction"/>
      <TaskNode identifier="stop" task="stop"/>
    </NodeList>
    <TransitionList>
      <Transition source="start" target="validateInput"/>
      <Transition source="validateInput" target="calculateRisk"/>
      <Transition source="calculateRisk" target="determineAction"/>
      <Transition source="determineAction" target="stop"/>
    </TransitionList>
  </body>
  <uuid>1a2b3c4d-5e6f-7890-abcd-ef1234567890</uuid>
</ilog.rules.studio.model.ruleflow:RuleFlow>
```

Ruleflow elements:
- **StartTask / StopTask**: Entry and exit points (exactly one each)
- **RuleTask**: Executes rules from specified packages
  - `packages`: Comma-separated list of rule packages to include
  - `algorithm`: `sequential` (evaluates all), `fastpath` (optimized Rete), `dynamic` (Decision Center priority)
  - `ordering`: `literal` (file order) or `dynamic` (priority-based)
- **SubflowTask**: Calls another ruleflow
- **BranchTask / ForkTask**: Conditional/parallel branching
- **Transitions**: Connect nodes, optionally with conditions

### Deployment Configuration (`.dep`)

RuleApp deployment descriptors define how rule projects get packaged for RES:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ilog.rules.studio.model.deployment:Deployment
    xmlns:ilog.rules.studio.model.deployment="http://ilog.rules.studio.model.deployment"
    name="MyRuleApp">
  <ruleAppName>MyRuleApp</ruleAppName>
  <ruleAppVersion>1.0</ruleAppVersion>
  <managedXom>true</managedXom>
  <rulesets>
    <ruleset>
      <rulesetName>MainRuleset</rulesetName>
      <rulesetVersion>1.0</rulesetVersion>
      <ruleProject>/MyRuleProject</ruleProject>
      <ruleflow>resources/mainpackage/MainFlow</ruleflow>
    </ruleset>
  </rulesets>
</ilog.rules.studio.model.deployment:Deployment>
```

---

## Common Tasks

### Reading an existing ODM project

When asked to read/understand an ODM project:

1. **Start with the BOM** (`bom/*.bom`) — this reveals the domain model and how Java classes map to business terms
2. **Check ruleflows** (`resources/**/*.rfl`) — these show the execution architecture
3. **Read the rules** (`rules/**/*.brl`, `*.dta`, `*.trl`) — now you understand the vocabulary from step 1
4. **Check deployment** (`deployment/*.dep`) — see how it's packaged for RES

### Creating new BAL rules

When creating a `.brl` file:

1. Always generate a fresh UUID (use `uuidgen` or a UUID v4 generator)
2. Match the package path to the directory structure under `rules/`
3. Use BOM verbalizations exactly as defined — don't invent new phrasings
4. Validate that all referenced BOM members exist in the `.bom` file
5. Set appropriate priority (lower number = higher priority, default is 0)

### Creating new decision tables

When creating a `.dta` file:

1. Condition expressions must use BOM verbalizations
2. Action expressions use `<a type>` placeholders for the parameterized parts
3. Ensure condition ranges don't have gaps or overlaps unless intentional
4. Generate a fresh UUID

### Modifying ruleflows

When editing `.rfl` files:

1. Every TaskNode must reference an existing Task in TaskList
2. Every Transition must reference valid source/target node identifiers
3. There must be exactly one StartTask and one StopTask
4. All paths must eventually reach the StopTask (no dead ends)
5. When adding a RuleTask, ensure the referenced package exists in the project

### XOM/BOM Java development

When working with XOM Java classes:

1. Follow JavaBean conventions: private fields + public getters/setters
2. Implement `Serializable` — RES requires it for remote execution
3. Use `@NotBusiness` annotation to hide methods from the BOM
4. Date fields should use `java.util.Date` (ODM's default) unless the project explicitly uses `java.time`
5. Collections should use `java.util.List` or `java.util.Collection` — BOM handles these natively
6. After changing XOM, the BOM must be updated to reflect new/changed/removed members

---

## Version-Specific Notes

For reference on version differences, read `references/version-notes.md`.

Key highlights:
- **8.9**: Baseline version, Eclipse-based Rule Designer, classic Decision Center
- **8.10**: Improved Decision Center UI, better Git integration, enhanced REST API
- **8.11+**: Container-native deployment (OpenShift/K8s), enhanced cloud support, Decision Runtime on Liberty

---

## Debugging Common Issues

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| "Unresolved BOM member" | BOM verbalization doesn't match BAL text exactly | Check spelling, case, and `{this}` placeholders in BOM |
| "Cannot resolve XOM type" | XOM JAR not in classpath or class renamed | Verify JAR is referenced, class name matches BOM entry key |
| Rule doesn't fire | Conditions never true, or wrong package in ruleflow | Check ruleflow `packages` attribute, verify test data |
| Decision table overlap warning | Two rows match the same input | Adjust range boundaries or add priority |
| Deployment fails on RES | Version conflict or missing XOM | Check RuleApp/ruleset versions, verify managed XOM |
| "Duplicate UUID" | Copy-pasted rule file without changing UUID | Generate new UUID for the copied rule |

---

## Important Reminders

- **UUIDs are mandatory and must be unique** across the project. Never reuse a UUID from another rule.
- **BOM verbalizations are the contract** between the model and the rules. Changing a verbalization breaks every rule that uses it.
- **Ruleflow package references are paths**, not Java packages. They use `/` separators and are relative to the `rules/` directory.
- **XML namespaces matter**. Each ODM file type uses a specific namespace URI. Getting it wrong makes the file unreadable by Rule Designer and Decision Center.
- When in doubt about a specific XML schema detail, read the actual file from the project rather than guessing — ODM's XML schemas are not publicly documented in full detail, and real project files are the best reference.
