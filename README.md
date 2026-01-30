# OpenBio Skills

Agent Skills for OpenBio - Unified biological data APIs and computational tools.

## Installation

```bash
bunx skills add https://github.com/openbio-ai/skills --skill openbio
```
or

```bash
npx skills add https://github.com/openbio-ai/skills --skill openbio
```

This installs the OpenBio skill to all detected agents (Claude Code, Cursor, Cline, etc.) via symlink.

## What's Included

The OpenBio skill provides domain-specific knowledge for using the OpenBio API, which offers 60+ biological tools:

- **Protein Structures** - PDB, PDBe, AlphaFold, UniProt
- **Literature Search** - PubMed, arXiv, bioRxiv, OpenAlex
- **Genomics** - Ensembl, ENA, GWAS, GEO
- **Cheminformatics** - RDKit, PubChem, ChEMBL
- **Molecular Biology** - PCR, Gibson/Golden Gate assembly
- **Structure Prediction** - Boltz, Chai, ProteinMPNN
- **Pathway Analysis** - KEGG, Reactome, STRING
- **Clinical Data** - ClinicalTrials.gov, ClinVar, FDA, Open Targets

## Authentication

All API requests require `OPENBIO_API_KEY` environment variable.

```bash
export OPENBIO_API_KEY=your_api_key_here
```

**Base URL**: `https://openbio-api.fly.dev/`

## Usage

Once installed, the skill is automatically available to your AI agents. Reference the skill when working with biological data:

```
Use the OpenBio skill to search for protein structures...
```

The skill includes detailed documentation in `rules/` for each tool category.

## Manual Installation

If you prefer manual installation, copy `skills/openbio/` to your agent's skills directory:

```bash
# Clone the repository
git clone https://github.com/openbio-ai/skills.git openbio-skills

# Copy to your skills directory (example for Claude Code)
cp -r openbio-skills/skills/openbio ~/.claude/skills/
```
