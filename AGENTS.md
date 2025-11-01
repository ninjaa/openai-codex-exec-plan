# Test driven development

Almost always write tests first, and have me approve them. Unit tests like that, integration tests ask my permission but we generally want those also unless a service being tested has side effects that are unsafe (I can help decide this).

# ExecPlans

When writing complex features or significant refactors, use an ExecPlan (as described in .agents/_template/PLANS.md) from design to implementation. Write new plans to the .agents dir, Place any temporary research, clones, etc., in a .gitignored subdirectory of .agents. But for permanent features, open a new subdir under .agents, so for example for the orchestrator, we would use .agents/orchestrator/PLAN.md ... start by doing `cp -r .agents/_template .agents/orchestrator`. You need to remove the preamble of each plan, do not copy blindly. And if you need to take a different approach than TDD just for a particular plan, put that clearly in the plan.
