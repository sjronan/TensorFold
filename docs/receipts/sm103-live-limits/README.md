# Development live resource audit

A full sm_103 contract at adb3a9729 completed all numerical and served-state
suites, then failed with 178 missing pinned usage records for other chips.
The same run passed 951 current-device launch checks and both driver-limit
comparisons; both resource controls were red. Its overall exit was 1, with
618 numerical kernels passing, eight registered findings and all 411 planted
numerical controls caught. This failed run is not a qualification pass.

`live()` previously added the five release targets' pinned-record audit to
every full device run. The declared sm_103 and sm_110 development targets
have no pinned columns; they audit compiled resources instead. Missing
release receipts therefore made a valid development-device audit fail.

The fix selects pinned records only for their release targets and keeps the
existing strict behavior for unknown devices. A declared development target
reports that the pinned release matrix is outside its coverage, then still
checks every applicable current-device launch against real function
attributes, block limits and both planted resource controls. Numerical gates,
known findings, tolerances, launch registrations and release records are
unchanged. Release-device runs still require the complete pinned matrix.

The host regression uses a real record with five missing release receipts.
Before the fix, the development selection returned five problems where the
test requires no pinned-column result. After the fix, sm_103/110 select the
live audit, while all five release devices and an unknown device still
reject the missing matrix. A partially filled release matrix still fails.
Valid compiled usage passes the development limits check; a launch beyond
its compiled bound still fails.

Run the regression without a GPU:

```sh
zig build test-contract-images-host -Dsm=103 -Doptimize=safe
```

A fresh full unfiltered device replay is required after this source change.
No partial counts from the failed run may fill that replay's SUMMARY.
