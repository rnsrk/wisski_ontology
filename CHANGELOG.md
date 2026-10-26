# Changelog

All notable changes to the WissKI default ontology.
Published versions: `https://wiss-ki.eu/sites/default/files/ontology/wisski/<version>.owl`

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow [Semantic Versioning](https://semver.org/).
Class and property names are given without namespace; `crm:` = `http://www.cidoc-crm.org/cidoc-crm/`, `crmdig:` = `http://www.cidoc-crm.org/extensions/crmdig/`, `lrmoo:` = `http://iflastandards.info/ns/lrm/lrmoo/`.

## [2.7.0] – unreleased

Annotations get a semantic model based only on CIDOC CRM and CRMdig, so that annotations drawn in the IIIF viewer (Mirador) can be stored as WissKI individuals and queried in the triplestore.

```
Annotation (D29) ── P1 is identified by ──▶ UUID ── P3 has note ──▶ "3f2b…"
                 ├─ P190 has symbolic content ──▶ "{…JSON…}"                       serialisation
                 ├─ P106 is composed of ──▶ Annotation_Text (E33) ── P3 has note ──▶ text
                 ├─ L43 annotates ──▶ Annotated_Area (D35 Area)
                 │                     ├─ P190 has symbolic content ──▶ "<svg …>"
                 │                     └─ L49 is primary area of ──▶ Image_File ── P1 is identified by ──▶ URL ── P3 has note ──▶ file URL
                 ├─ P67 refers to ──▶ Canvas ── P1 is identified by ──▶ URL ── P3 has note ──▶ canvas URL
                 └─ L48i was annotation created by ──▶ D30 Annotation Event
                                                        ├─ P14 carried out by ──▶ E21 Person ── P1 is identified by ──▶ E41 Appellation ── P3 has note ──▶ creator
                                                        └─ P4 has time-span ──▶ E52 ── P82 at some time within ──▶ "2026-10-14T20:57:00Z"
```

Only CIDOC CRM and CRMdig properties are used. Values are given with `P3 has note`, except the creation time (`P82 at some time within`) and the symbolic content of the annotation (JSON) and of its area (SVG), which use `P190 has symbolic content`. The area keeps only the SVG geometry: a second `P190` value on the same class could not be told apart from it, and a rectangle (`xywh=…`) is an SVG `rect`.

The geometry of an area is not a separate entity: `D35 Area` is a symbolic object (via `D1` ⊑ `E73` ⊑ `E90`), and its SVG geometry is its symbolic content, like the coordinates of a METS `AREA`. The model is limited to what is needed to store and find an annotation: identifier, serialisation, text, area and image, canvas, creator and creation time. Everything else (motivation, styling, replies) stays in the serialisation. Creator and creation time use the CRMdig `D30 Annotation Event` directly; no new class is needed. The creator is a plain `E21 Person` with a name, not linked to other person records.

The annotation refers to the IIIF canvas it was drawn on (`P67 refers to` `Canvas`): one page of a manifest, identified by the canvas document URL and reused for that address. The area still belongs to the image file (`L49 is primary area of` `Image_File`). Canvas size equals image size, so the coordinates are pixels of that page.

### Added
- `Annotated_Area` ⊑ `crmdig:D35_Area`: the annotated area of an image (`L49 is primary area of` `Image_File`).
- `Annotation_Text` ⊑ `crm:E33_Linguistic_Object`: the text of an annotation.
- `Canvas` ⊑ `Digital_Media`: one page of an IIIF manifest, not the image file (`Annotation` `P67 refers to`). Identified by its canvas URL; the same individual is reused for the same address.
- `Cultural_Heritage_Item_Type` ⊑ `crm:E55_Type`: the type of a Cultural Heritage Item (object type / object appellation via `P2 has type`). Replaces the former path that used `Authority_Data` as type target before 2.6.0.
- Ontology header: annotations listed among the covered areas.

### Changed
- `Annotation` is now a subclass of `crmdig:D29_Annotation_Object` instead of `crm:E73_Information_Object` (not breaking: `D29` ⊑ `E89` ⊑ `E73`). Comment describes the intended paths; German comment added.

## [2.6.0] – unreleased

Authority data is remodelled: an authority record is no longer a type but a document that is part of an authority file.

```
Entity (E1) ── P70i is documented in ──▶ Authority_Data (E31 Document)
                                           ├─ P1 is identified by ──▶ Authority_Identifier (E42)   "118540238"
                                           ├─ P1 is identified by ──▶ URL (E42)                    https://d-nb.info/gnd/118540238
                                           ├─ P1 is identified by ──▶ Authority_Term (E41)         authorised form
                                           ├─ P1 is identified by ──▶ Alternative_Name (E41)       variant forms
                                           ├─ P2 has type ──────────▶ Authority_Data_Type (E55)    e.g. GND entity type
                                           ├─ P16i was used for ──▶ Authority_Data_Retrieval (D12) ── P4 has time-span ──▶ E52
                                           │                                                     └─ L15 has sender ──▶ D8 Digital Device (service URL)
                                           └─ P106i forms part of ──▶ Authority_File (E32 Authority Document)   GND, AAT, GeoNames …
                                                                        ├─ P1 is identified by ──▶ Preferred_Name / Acronym / URL (website)
                                                                        └─ P94i was created by ──▶ E65 Creation ── P14 carried out by ──▶ E74 Group (provider, e.g. DNB)
Shortcut: Entity ── P71i is listed in ──▶ Authority_File
```

### Added
- `has_digital_carrier` ⊑ `crm:P130_shows_features_of` (`Digital_Media` → `Media_File`) and its inverse `is_digital_carrier_of`. CRMdig only has `L19 stores` for a physical `D13 Digital Information Carrier`; the file that stores a digital object is not that. Replaces OntPreHer3D `R19` / `R19i`.
- `Authority_File` ⊑ `crm:E32_Authority_Document`: the authority file, thesaurus or vocabulary as a whole (GND, AAT, TGN, ULAN, GeoNames, Wikidata, VIAF …).
- `Authority_Identifier` ⊑ `crm:E42_Identifier`: the identifier of a record within its authority file.
- `Authority_Data_Retrieval` ⊑ `crmdig:D12_Data_Transfer_Event`: retrieval of a record from the provider's service (date, source endpoint).
- German labels for `Authority_Data`, `Authority_Data_Type`, `Authority_Term`.
- Ontology header: `owl:priorVersion`, `dcterms:license` (CC BY 4.0) and an extended `rdfs:comment` (en, de) describing purpose and covered areas.

### Changed
- Removed imports of OntPreHer3D 2.1.28 and OntSciDoc3D 2.0.2. `M45 File Format` is already `Media_File_Format` ⊑ `crm:E55_Type`.
- **Breaking:** `Authority_Data` is now a subclass of `crm:E31_Document` instead of `crm:E55_Type`. Instances are authority records that document an entity (`P70i is documented in`); they are no longer used as `P2 has type` targets. See *Migration* below.
- `Authority_Term` is clarified as the authorised (preferred) form of a name in an authority record; variant forms use `Alternative_Name`.
- `Authority_Data_Type` is clarified as the type of a record within its authority file (e.g. GND entity type, GeoNames feature code, Getty vocabulary).
- `Getty_Vocabulary_Type`: comment clarified; AAT, TGN and ULAN as a whole are instances of `Authority_File`.

### Fixed
- `Audio_File`: English label was "Image", now "Audio".
- `Comment`: German label "Kommentart" → "Kommentar".
- `Digital_Media_Creation`: German label "Digitale Medieneerstellung" → "Digitale Medienerstellung".
- `Digital_Media_Type`: German label "Digitales Medientyp" → "Digitaler Medientyp".
- `Date_Operator`: label "Datumsoperator" was tagged `@en`, now `@de`.

### Migration
Pathbuilder steps that used OntPreHer3D can be pointed at the WissKI ontology:

- `…/OntPreHer3D/M45_File_Format` → `Media_File_Format` (the path stays `Media_File` — `P2 has type` — `Media_File_Format` — `P1 is identified by` — `E41 Appellation`).
- `…/OntPreHer3D/R19_has_digital_carrier` → `has_digital_carrier`.
- `…/OntPreHer3D/R19i_is_digital_carrier_of` → `is_digital_carrier_of`.

Data that linked an entity to an `Authority_Data` instance with `P2 has type` can be moved to `P70i is documented in`:

```sparql
PREFIX crm: <http://www.cidoc-crm.org/cidoc-crm/>
PREFIX wisski: <http://wiss-ki.eu/ontology/default/>
DELETE { ?entity crm:P2_has_type ?record . ?record crm:P2i_is_type_of ?entity . }
INSERT { ?entity crm:P70i_is_documented_in ?record . ?record crm:P70_documents ?entity . }
WHERE  { ?record a wisski:Authority_Data . ?entity crm:P2_has_type ?record . }
```

Pathbuilder paths that pass through `Authority_Data` via `P2 has type` must be rebuilt accordingly.

### Known issues
- `Date_Period` refers to `has_begin_date` / `has_end_date`, which are not defined in the ontology.
- `Data_Record` is a subclass of `crmdig:D9_Data_Object`, which CRMdig restricts to results of digital measurements (D9 ⊑ E54 Dimension).
- `Date_Operator` has an empty comment.

## [2.5.0]

### Added
- Media file classes ⊑ `Media_File`: `Image_File`, `Audio_File`, `Video_File`, `Document_File`, `3D_Model_File`, `Generic_File`.
- `Actor_Relation` ⊑ `crm:E13_Attribute_Assignment`: relations between actors (kinship, organisational structure).
- `Preservation` ⊑ `crm:E4_Period` and `Preservation_Type` ⊑ `crm:E55_Type`.
- `Reference` ⊑ `crm:E33_Linguistic_Object`: a reference to a source cited for an entity.
- `Reference_Location` ⊑ `crm:E42_Identifier`: the pinpoint within a cited source (page, entry, timestamp).
- `Type_Assignment_Level` ⊑ `crm:E55_Type`.

### Removed
- `Image` (⊑ `crm:E36_Visual_Item`), superseded by `Image_File`.

## [2.4.0]

### Added
- Imports: CRMinf 1.2.1, CRMsci 3.0.1, OntPreHer3D 2.1.28, OntSciDoc3D 2.0.2.
- Datatype properties ⊑ `crm:P190_has_symbolic_content` (domain `crm:E90_Symbolic_Object`, range `xsd:string`): `symbolic_image_content`, `symbolic_audio_content`, `symbolic_video_content`, `symbolic_document_content`, `symbolic_3d_content`.

## [2.3.0]

### Added
- Activities: `Action` and `Analysis` ⊑ `crm:E7_Activity`; `Activity_Type` ⊑ `crm:E55_Type` with subclass `Action_Type`.
- Events: `Event_Main_Type` ⊑ `Event_Type`, `Event_Sub_Type` ⊑ `Event_Main_Type`; `Phase` ⊑ `crm:E4_Period`.
- Assignments ⊑ `crm:E13_Attribute_Assignment`: `Cultural_Heritage_Item_Assignment`, `Object_Assignment`, `Custom_Term_Payload`; types `Actor_Assignment_Scope`, `Object_Assignment_Type`.
- LRMoo: `Cultural_Heritage_Work` ⊑ `lrmoo:F1_Work`, `Cultural_Heritage_Expression` ⊑ `lrmoo:F2_Expression`.
- Digital: `Digital_Media` ⊑ `crmdig:D1_Digital_Object`; `Digital_Media_Creation`, `Data_Record_Creation` ⊑ `crmdig:D2_Digitization_Process`; `Digital_Media_Type`, `Media_File_Format` ⊑ `crm:E55_Type`.
- Things: `Sample`, `Tool` ⊑ `crm:E22_Human-Made_Object`; `Version_Number` ⊑ `crm:E99_Product_Type`.
- Object properties `was_directed_at` (⊑ `crm:P15_was_influenced_by`, E7 → E70) and its inverse `was_target_of`.

### Removed
- `Media_Type`, superseded by `Digital_Media_Type`.

## [2.2.0]

The published file declares `owl:versionInfo` and `owl:versionIRI` as 2.3.0; only the `rdfs:label` says 2.2.0.

### Added
- Import: LRMoo 1.1.1.
- Provenance periods ⊑ `crm:E4_Period`: `Ownership`, `Custody`, `Stay`, `Date_Period`; with types `Custody_Type`, `Stay_Type`, `Date_Period_Type`.
- Event types: `Acquisition_Type`, `Move_Type`, `Transfer_of_Custody_Type`, `Place_Type`.
- `Organised_Event` ⊑ `crm:E5_Event` and `Organised_Event_Type`.
- `Informational_Source` ⊑ `lrmoo:F3_Manifestation` and `Informational_Source_Type`.
- `Bibliographical_Info` ⊑ `crm:E33_Linguistic_Object`.

### Changed
- `Event_Type`: German label "Ereignis Typ" → "Ereignistyp", comment clarified.

### Removed
- `Resource` and `Resource_Type`, superseded by `Informational_Source` and `Informational_Source_Type`.
- Object properties `begins_with` and `ends_with`.

## [2.1.0]

### Added
- Subclasses of `Authority_Data_Type`: `GND_Entity_Type`, `Geonames_Feature_Code_Type`, `Getty_Vocabulary_Type`, `Custom_Term_Type`.

## [2.0.0]

### Changed
- **Breaking:** stable namespace `http://wiss-ki.eu/ontology/default/` instead of a versioned one; the version is given by `owl:versionIRI`.
- **Breaking:** imports the WissKI copies of CIDOC CRM 7.1.3 (`http://wiss-ki.eu/ontology/crm/7.1.3/`) and CRMdig 4.0 instead of Erlangen CRM 240307; CRM references use `http://www.cidoc-crm.org/cidoc-crm/`.
- **Breaking:** renamed `PreferredName` → `Preferred_Name`, `SpatialCoordinates` → `Spatial_Coordinates`.
- `Annotation` is now a subclass of `crm:E73_Information_Object` instead of `crm:E13_Attribute_Assignment`.
- Scope widened from "museal" to "cultural heritage" context.
- `skos:prefLabel` / `skos:notation` introduced; comments carry language tags.

### Added
- Names: `Abbreviation`, `Acronym`, `Alternative_Name`, `Custom_Term`, `Authority_Term` ⊑ `crm:E41_Appellation`; `Inventory_Identifier` ⊑ `crm:E42_Identifier`.
- Authority data: `Authority_Data`, `Authority_Data_Type` ⊑ `crm:E55_Type`.
- Assignments ⊑ `crm:E13_Attribute_Assignment`: `Actor_Assignment`, `Date_Assignment`, `Media_Assignment`, `Period_Assignment`, `Title_Assignment`.
- Activities and events: `Item_History_Activity`, `Occupation` ⊑ `crm:E7_Activity`; `Begin`, `End` ⊑ `crm:E5_Event`.
- Things and media: `Cultural_Heritage_Item` ⊑ `crm:E22_Human-Made_Object`; `Data_Record`, `Media_File` ⊑ `crmdig:D9_Data_Object`; `Resource` ⊑ `crm:E73_Information_Object`.
- Types ⊑ `crm:E55_Type`: `Actor_Type`, `Assurance`, `Data_Record_Type`, `Date_Operator`, `Dimension_Type`, `Event_Type`, `Keyword`, `Media_Type`, `Organisation_Type`, `Precision`, `Resource_Type`, `Role`, `Visibility`.

### Removed
- `Canvas`, `Citation`, `HumanMadeObjectInformation`, `Image_Annotation`, `JSON`, `SVG`.

## [1.2.0]

### Changed
- **Breaking:** versioned namespace `http://wiss-ki.eu/ontology/1.2.0/`.

### Added
- `Annotation` ⊑ `crm:E13_Attribute_Assignment` and `Image_Annotation` ⊑ `Annotation`.
- `Image` ⊑ `crm:E36_Visual_Item` and `Canvas` ⊑ `Image`.
- `Data_Object` ⊑ `crm:E73_Information_Object` with `JSON` and `SVG`.
- `Section` ⊑ `crm:E53_Place`, `UUID` ⊑ `crm:E42_Identifier`.

## [1.1.0]

### Changed
- **Breaking:** versioned namespace `http://wiss-ki.eu/ontology/1.1.0/`.
- CIDOC CRM is no longer embedded; imports Erlangen CRM 240307 (CIDOC CRM 7.1.3).

### Added
- Object properties from `crm:E52_Time-Span` to `Date`: `begins_with`, `ends_with`, `has_some_date_within`.

## [1.0.0]

Initial release. Namespace `http://wiss-ki.eu/ontology/`, CIDOC CRM 7.1.2 embedded.

### Added
- `Comment`, `Citation`, `Description`, `Note` ⊑ `crm:E33_Linguistic_Object`; `Source` ⊑ `crm:E31_Document`.
- `Date` ⊑ `crm:E52_Time-Span` with `Day`, `Month`, `Year`.
- `PreferredName`, `SpatialCoordinates` ⊑ `crm:E41_Appellation`; `URL` ⊑ `crm:E42_Identifier`.
- `HumanMadeObjectInformation` ⊑ `crm:E13_Attribute_Assignment`; `Status` ⊑ `crm:E55_Type`.

[2.7.0]: https://wiss-ki.eu/sites/default/files/ontology/wisski/2.7.0.owl
[2.6.0]: https://wiss-ki.eu/sites/default/files/ontology/wisski/2.6.0.owl
[2.5.0]: https://wiss-ki.eu/sites/default/files/ontology/wisski/2.5.0.owl
[2.4.0]: https://wiss-ki.eu/sites/default/files/ontology/wisski/2.4.0.owl
[2.3.0]: https://wiss-ki.eu/sites/default/files/ontology/wisski/2.3.0.owl
[2.2.0]: https://wiss-ki.eu/sites/default/files/ontology/wisski/2.2.0.owl
[2.1.0]: https://wiss-ki.eu/sites/default/files/ontology/wisski/2.1.0.owl
[2.0.0]: https://wiss-ki.eu/sites/default/files/ontology/wisski/2.0.0.owl
[1.2.0]: https://wiss-ki.eu/sites/default/files/ontology/wisski/1.2.0.owl
[1.1.0]: https://wiss-ki.eu/sites/default/files/ontology/wisski/1.1.0.owl
[1.0.0]: https://wiss-ki.eu/sites/default/files/ontology/wisski/1.0.0.owl
