# lex-cognitive-plasticity

Neural plasticity model for LegionIO cognitive agents. Pathways have strength and plasticity; strengthening is scaled by current plasticity, which itself decreases as pathways mature. Critical period mode applies a 2.5× multiplier for accelerated learning.

## What It Does

- Five pathway types: `synaptic`, `structural`, `functional`, `homeostatic`, `metaplastic`
- Strength: how established the pathway is (0.0–1.0)
- Plasticity: how receptive to change (0.1–1.0, decays with each activation)
- `strengthen_pathway`: effective gain = amount × plasticity (× 2.5 in critical period)
- `weaken_pathway`: effective reduction = amount × plasticity
- `rejuvenate_pathway`: re-open a mature pathway to change
- Critical period: engine-wide 2.5× boost on all strengthening
- Auto-prune: weakest pathway removed when capacity (300) is reached
- `prune_weak_pathways`: explicit pruning of pathways below strength threshold (0.15)

## Usage

```ruby
# Create a pathway
result = runner.create_pathway(label: 'pattern_recognition_loop',
                                pathway_type: :synaptic, initial_strength: 0.3)
pid = result[:pathway][:id]

# Enter critical period (imprint window)
runner.enter_critical_period
# => { success: true, critical_period: true }

# Strengthen with 2.5× boost
runner.strengthen_pathway(pathway_id: pid, amount: 0.08)
# => { success: true, pathway: { strength: 0.40, plasticity: 0.49, activation_count: 1, ... } }

# Exit critical period
runner.exit_critical_period

# Rejuvenate a mature pathway to re-open it to change
runner.rejuvenate_pathway(pathway_id: pid, amount: 0.1)

# Prune weak pathways
runner.prune_weak_pathways
# => { success: true, pruned_count: 0 }

# Report
runner.plasticity_report
# => { success: true, report: { total_pathways: 1, critical_period: false, average_strength: 0.4, ... } }
```

## Development

```bash
bundle install
bundle exec rspec
bundle exec rubocop
```

## License

MIT
