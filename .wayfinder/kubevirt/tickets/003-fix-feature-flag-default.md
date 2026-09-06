# Fix Feature Flag Default (G-08)

**Label:** `wayfinder:task`  
**Status:** Closed  
**Blocked by:** (none)

## Question

Change `OADP_ENABLE_KUBEVIRT` default from `True` to `False` in `environment_variables.py:60`. One-line fix for safe rollout — feature must be explicitly opted into per-environment.

Resolve: make the change, confirm unit tests still pass.

## Resolution

Already implemented in commit `6c8379e5` ("fix(kubevirt): safe-rollout defaults and plugin arg validation (G-08/G-09)").

Verified: `environment_variables.py:60` reads `get_boolean_from_environment('OADP_ENABLE_KUBEVIRT', False)`. Unit test `test_kubevirt_scenario_rejected_when_flag_off` confirms rejection behavior.
