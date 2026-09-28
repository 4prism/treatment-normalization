# Treatment Record Normalization

Every dental clinic names procedures differently. The same treatment is written `CR`
in one clinic, `크라운` in another, `Crown세팅` in a third. Appointment fields often
hold **several procedures in a single line**, such as `#26 cr prep or 근관`.

This repository is about the data mining that pulls those fragmented free-text clinical
records onto **one common basis before any of it reaches a model**.

## The problem

There is a wall you always hit before clinical data can be used for training.

| Problem | What it looks like |
|---|---|
| **Notation drift** | The same procedure is a different string at every clinic. Distinct spellings grow with the number of clinics |
| **Several procedures in one field** | Joined with `or`, `/`, or a space. Counting the field as one entry skews every statistic |
| **Abbreviations and typos** | `imp`, `Imp`, `임플`, `임플란트` all mean the same thing |
| **Billing codes don't line up** | Claim codes are precise but **they are not the unit of care**. You cannot reconstruct the treatment from the code alone |

Aligning the notation of a single clinic by hand is feasible. The problem is that
**the work restarts from zero for every clinic you add**. Normalization exists to end
that repetition.

## Approach

```
raw text  →  ① split  →  ② normalize  →  ③ map  →  normalized record
"#26 cr prep or 근관"   tooth / procedure   unify spelling   link to a base concept
```

1. **Split** — separate a mixed field into individual procedures. Delimiters are not
   consistent, so a plain `split` is not enough; the order of tooth number, site and
   procedure has to be read together.
2. **Normalize** — fold case, whitespace and abbreviations into one spelling.
   Rules handle most of this stage.
3. **Map** — connect the normalized spelling to a **base concept**. Clinic-specific
   notation is absorbed here, so onboarding a new clinic only adds mappings.

## Design constraints we hold to

- **The original is never rewritten.** Normalized output goes into a separate layer and
  the source text stays as it is. Clinical records are legal records, so we apply no
  irreversible transformation to them.
- **When unsure, leave it empty.** A spelling that does not map confidently is left
  unmatched rather than guessed. A wrong normalization is worse than a missing one —
  because the next stage runs on it as if it were true.
- **One place decides.** If the same spelling is interpreted in two places, the two will
  drift apart.

## Status

Early public stage. The design and the approach are written first; implementation and
measured results follow.

---

4Prism · Software for dental clinic operations
