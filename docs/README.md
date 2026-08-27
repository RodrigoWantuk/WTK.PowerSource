# Technical documentation

This directory contains the engineering specifications, design notes, validation records, and revision documents for WTK.PowerSource.

## Current baseline

- [`WTK_PowerSource_Especificacao_Tecnica_Rev_B4.pdf`](WTK_PowerSource_Especificacao_Tecnica_Rev_B4.pdf) — consolidated Rev.B.4 pre-prototype technical specification.

Rev.B.4 is the formal PDF baseline currently stored with the project. The repository `README.md` and subsequent committed engineering notes may supersede individual assumptions as the design evolves; a new PDF revision should be generated when enough changes accumulate to justify a new frozen baseline.

## Documentation rules

- Treat calculated values, datasheet-derived values, bench-tuned values, and open design decisions as different categories.
- Do not promote a target into a guaranteed specification until it has passed the corresponding bench-validation gate.
- Every released PCB revision must reference the matching documentation revision.
- Record changes that affect safety, isolation, SOA, thermal behavior, current limits, series operation, or protection sequencing.
- Keep obsolete architecture discussions out of the active baseline; historical documents may remain for traceability when clearly marked as superseded.

## Planned additions

As the project moves from architecture to implementation, this directory should grow to include documents such as:

```text
docs/
├── README.md
├── WTK_PowerSource_Especificacao_Tecnica_Rev_B4.pdf
├── schematic-notes/
├── bring-up/
├── validation/
├── calibration/
└── revisions/
```

Suggested future topics include:

- schematic sheet/interface specification;
- detailed power-stage calculations;
- protection thresholds and coordination;
- isolation audit;
- TIP36C SOA validation;
- preregulator characterization;
- ripple/noise measurements;
- current-measurement calibration;
- reverse-power/backfeed tests;
- thermal characterization and fan curves;
- series/symmetric-mode validation;
- firmware/hardware interface contract.
