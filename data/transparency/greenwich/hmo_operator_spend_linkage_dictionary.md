# Greenwich Spend Linkage Data Dictionary

## Raw vs Derived
- `raw`: value is copied from source spend files (possibly type-normalized only)
- `derived`: value is computed during cleaning/linkage

## `payments_over_500_staging.parquet`
- `supplier_name` (`raw`): supplier/payee name from council transparency file.
- `amount_raw` (`raw`): original amount text before numeric coercion.
- `payment_date_raw` (`raw`): original payment date text before parsing.
- `department` (`raw`): source department/service area column when present.
- `description` (`raw`): source description/category column when present.
- `source_url` (`raw`): original file URL used.
- `source_file` (`derived`): local filename used for ingestion.
- `amount` (`derived`): parsed numeric GBP value.
- `payment_date` (`derived`): parsed datetime value.
- `supplier_norm` (`derived`): normalized supplier key used for matching.

## `hmo_operator_spend_linkage_rows.parquet`
- `council` (`derived`): council slug, currently fixed to `greenwich`.
- `canonical_entity_id` (`derived`): matched HMO canonical holder key from TASK-314 mapping.
- `holder_norm` (`derived`): normalized canonical holder key used for matching.
- `match_type` (`derived`): `exact_norm` or `fuzzy_norm`.
- `match_confidence` (`derived`): match score (exact=100; fuzzy from RapidFuzz).
- `tier` (`derived`): confidence tier (`1` exact normalized match, `2` fuzzy >=90).
- `is_housing_context` (`derived`): True if department/description contains housing/support keywords.
- `ta_signal` (`derived`): True when row text matches temporary-accommodation proxy terms.
- `support_signal` (`derived`): True when row text matches support/care proxy terms.
- `exempt_strong_signal` (`derived`): True only when explicit exempt/specified-accommodation terms appear.
- `inferred_relationship` (`derived`): pragmatic interpretation bucket (`probable_ta`, `probable_support`, `possible_exempt`, `no_signal`).
- `inference_confidence` (`derived`): interpretation confidence (`high`/`medium`/`low`) after combining text-signal quality with linkage tier.
- `inference_caveat` (`derived`): mandatory caveat text for narrative-safe usage.
- `payment_year` (`derived`): year extracted from parsed payment date.
- plus staging columns above for row lineage.

## `hmo_operator_spend_linkage.parquet` (operator summary)
- `council` (`derived`): council slug.
- `canonical_entity_id` (`derived`): matched HMO operator key.
- `tier` (`derived`): confidence tier bucket for the grouped rows.
- `total_amount_gbp` (`derived`): summed `amount` for grouped rows.
- `match_count` (`derived`): number of matched payment rows.
- `max_confidence` (`derived`): highest match confidence within group.
- `housing_context_hits` (`derived`): number of grouped rows where `is_housing_context=True`.

## `hmo_operator_spend_linkage_yearly.parquet` (year summary)
- `council` (`derived`): council slug.
- `canonical_entity_id` (`derived`): matched HMO operator key.
- `payment_year` (`derived`): year of payment date (nullable if date missing/unparseable).
- `total_amount_gbp` (`derived`): yearly summed amount.
- `match_count` (`derived`): yearly matched payment row count.
- `housing_context_hits` (`derived`): yearly housing-context row count.
- `best_tier` (`derived`): best confidence tier observed that year (`1` better than `2`).
- `max_confidence` (`derived`): highest row-level confidence that year.

## `hmo_operator_spend_linkage_explainer.parquet` (narrative-safe operator summary)
- `council` (`derived`): council slug.
- `canonical_entity_id` (`derived`): matched HMO operator key.
- `inferred_relationship` (`derived`): interpretation bucket from row-level rules.
- `inference_confidence` (`derived`): confidence label for interpretation bucket.
- `total_amount_gbp` (`derived`): summed amount for the interpretation bucket.
- `match_count` (`derived`): matched row count in the bucket.
- `housing_context_hits` (`derived`): count of housing-context supporting rows.
- `best_tier` (`derived`): best supplier-linkage tier observed in bucket.

## `hmo_operator_spend_linkage_explainer_yearly.parquet` (narrative-safe yearly summary)
- `council` (`derived`): council slug.
- `canonical_entity_id` (`derived`): matched HMO operator key.
- `payment_year` (`derived`): payment year.
- `inferred_relationship` (`derived`): interpretation bucket.
- `inference_confidence` (`derived`): confidence label.
- `total_amount_gbp` (`derived`): yearly summed amount.
- `match_count` (`derived`): yearly matched row count.
- `best_tier` (`derived`): best supplier-linkage tier observed that year.
