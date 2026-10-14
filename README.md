# gleif-groups

The GLEIF corporate-group graph as a self-contained Semvia semantic package:
who owns whom, and the legal entities on either end, bundled inside the archive.

Install it and the 482,824 GLEIF Level 2 relationship records and the 352,537
legal entities they connect become queryable over SPARQL, read from two Parquet
files the package carries. No credential, no bucket, and no network once it is
installed.

## Install

Paste `https://pkg.semvia.io/semvia/gleif-groups/@0.1.0` into Folio's
dependency drawer, or from a workspace:

```sh
sem add https://pkg.semvia.io/semvia/gleif-groups/@0.1.0
```

Either records the requirement in `sem.json`:

```json
{
  "requires": [
    {
      "@type": "https://pkg.semvia.io/semvia/core/semantic-package#PackageRequirement",
      "package": "https://pkg.semvia.io/semvia/gleif-groups/",
      "versionMinimum": "0.1.0"
    }
  ]
}
```

The archive is about 17 MB (16.9 MB gzipped, nearly all of it the two Parquet
files); the drawer shows bytes received while it downloads.

## The cut

| File | Rows | Bytes | What it holds |
| --- | ---: | ---: | --- |
| `data/relationship_records.parquet` | 482,824 | 5,869,457 | every Level 2 relationship record |
| `data/lei_records.parquet` | 352,537 | 11,099,769 | every Level 1 legal entity that is the start or end node of one of those records |

Every relationship is kept; an entity is kept when it takes part in at least one.
That is the whole corporate-group graph, closed under its edges: a walk up
`gleif:directlyConsolidatedBy+` from any entity in the cut reaches only entities
the full register would also give it. The relationship records name 352,541
distinct LEIs; the 4 the register itself does not carry are absent here too, so
their edges resolve to an IRI with no name, as they do in `gleif`. The
3,037,661 register entries with no relationship are
left out; [`gleif`](https://github.com/SemviaIO/gleif) reads the full register
from the public bucket.

Both files have the layout `gleif` uses: sorted on the key a query binds (`lei`;
`relationship_startnode_nodeid` then `relationship_endnode_nodeid`), row groups
of 300,000 rows, a page index, bloom filters on the key columns, and the sort
order recorded in the footer. Each map declares the same order with
`svcat:declaredSortOrder`.

`data/` is not committed. `tools/export-parquet.py` in `SemviaIO/gleif` writes
it from the Postgres golden copy; place its `gleif-groups/` output here before
running `sem package`.

## The model

The ontology and the relationship-type scheme are `gleif`'s, term for term. The
classes and predicates are `https://pkg.semvia.io/semvia/gleif/ontology#…`, so a
query written against `gleif` reads this package unchanged.

The two documents are copied, not imported. Importing would mean requiring
`gleif`, which brings its bucket-backed mappings over the same terms, and a
workspace holding both would answer every read twice. Each copied file says so
in its header.

```
schema/
  ontology.ttl             gleif's classes and predicates (copied)
  relationship-types.ttl   gleif's six relationship types and the lookup over them (copied)
  legal-entities.ttl       LeiRecords over data/lei_records.parquet
  relationships.ttl        Relationships over data/relationship_records.parquet
  related-entities.ttl     RelatedEntities: the typed child-to-parent edges
publish.ttl                the bucket this package is published to (not built, not packed)
```

The three mappings read the files by relative path (`rml:path "data/…"`), which
an installed package resolves against its own root. See `gleif`'s README for
the caveats worth knowing before you query; they all hold here.

## Licence and attribution

Everything Semvia authored here is CC0 1.0 Universal — see `LICENSE`.

The data is GLEIF's, and this package does redistribute it. See `NOTICE` for the
source, GLEIF's terms of use, and exactly what is included.
