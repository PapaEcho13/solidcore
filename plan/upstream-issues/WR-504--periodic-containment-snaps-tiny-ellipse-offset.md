#### WR-504. Periodic containment snaps a tiny ellipse arc's points to its start (#1518)

- **Severity:** Wrong result
- **Area:** Curves and intervals
- **Location:** `source/SMLib/src/SmExtent1d.cpp:1229`, `SmExtent1d::ContainsPeriodicValue`; reached by `SmEllipse::STEPInversion`.
- **What happens:** A mapped value near the top of the period is replaced by the interval minimum instead of moved down one period. A short ellipse arc in a negative angular domain consequently inverts interior points to its start.
- **Trigger:** Ellipse radii `70.9232`, `16`, angular domain `[-30, -29.99993]` degrees. At `-29.9999825` degrees the legacy result is `-30`, with a point round-trip gap of `1.1628620236000771e-5`.
- **How found:** Issue #1518; reproduced by `tiny_ellipse_periodic_inversion_preserves_offset` before the fix.
- **Confirmed:** Yes, pinned-reference C++ fixtures `issue_1518_tiny_arc_periodic_offset` in `geom.extent1d` and `curve.ellipse_p2`, both bit-identical to the default Rust path. The ellipse fixture constructs a wider arc and then sets its short STEP interval, respecting the adapter's minimum construction-domain guard.
- **Rust correction:** `Kernel::fixes.periodic_containment_preserve_offset` (off by default). `Extent1d::contains_periodic_value_with_kernel` subtracts one period and checks the lowered value against both interval ends with the existing tolerance. `Ellipse::step_inversion` uses this kernel-aware entry point. The legacy entry point and other callers retain pinned-reference behavior. Tests cover default compatibility, containment distances, boundaries, tolerance halos, multiple periods, nonperiodic inputs and ellipse point round trips. This does not claim that the related tube Boolean defects (#1517/#1519) are fixed.
