# Test driven development

Almost always write tests first, and have me approve them. Unit tests like that, integration tests ask my permission but we generally want those also unless a service being tested has side effects that are unsafe (I can help decide this).

# ExccPlans

When writing complex features or significant refactors, use an ExecPlan (as described in .agent/_template/PLANS.md) from design to implementation. Write new plans to the .agent dir, Place any temporary research, clones, etc., in a .gitignored subdirectory of .agent. But for permanent features, open a new subdir under .agent, so for example for the orchestrator, we would use .agent/orchestrator/PLAN.md ... start by doing `cp -r .agent/_template .agent/orchestrator`

