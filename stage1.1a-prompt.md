# Stage 1.1a — Extraction Prompt v3.7
 
 ### CONSTANTS 
 
 ``` 
 GLOSS_MIN    = 8   words   (≈ 11 nomic-embed tokens) 
 GLOSS_TARGET = 15  words   (≈ 21 nomic-embed tokens) 
 GLOSS_MAX    = 22  words   (≈ 30 nomic-embed tokens) 
 ``` 
 
 --- 
 
 You are extracting structured content from a single research paper for 
 ingestion into a semantic knowledge graph. 
 
 Extraction is a pure function of this paper, this prompt, and these 
 constants. You have no corpus, no research question, and no other paper. 
 Do not reach for a connection to another field: whether this paper's 
 content bridges to anything is decided downstream by geometry, over a 
 corpus you cannot see. Your only job is to read this paper faithfully and 
 name what is in it. 
 
 Read the entire paper, then extract three kinds of finding. Favor recall, 
 but apply the exclusion tests — they are not optional. 
 
 Ignore the reference list, acknowledgments, and closed-form formulas. 
 
 --- 
 
 ### TERMS 
 
 Any technical term, named object, or concept the paper treats as a unit. 
 Length is irrelevant: a single symbol (ξ, κ), a single word (hatpin), a 
 multi-word phrase (long-legged A), or a full clause naming a process 
 (*shift from hydration-dominated to carbonation-dominated phase 
 evolution*) all qualify equally. A clause qualifies when no word can be 
 removed without losing the referent. 
 
 **Exclusion tests.** Drop the term if any of these hold. 
 
 1. **The gloss cannot add content.** If the gloss only re-composes the 
    term's own words, the name already exhausts the thing. *noise 
    filtering* → "removal of retrieved content unrelated to the query" 
    restates the term; drop. *Scanning electron microscopy* → "imaging that 
    rasters a focused electron beam across a specimen surface" adds what 
    the name doesn't say; keep. This is a test of content, never of length. 
 2. **It is a measured quantity, not a concept.** *Degree of carbonation* 
    is defined as a concept and stays; *CO2 uptake* is a measurement and 
    does not. Figures survive only inside `paper_wording`. 
 3. **It is another term plus a qualifier.** *Theoretical CO2 uptake* is 
    *CO2 uptake* with a ceiling — one sense with a modifier, not two. 
    (*Apparent*, *absolute* and *relative density* pass: each names a 
    different measurement, not a bound on one.) 
 4. **It is apparatus.** A paper that builds a system names every part of 
    it; the parts list is scaffolding for the paper's own exposition and 
    grounds to nothing outside it. *sub-plan*, *config block*, *stage two 
    handler* — drop the whole inventory. 
 5. **It is research boilerplate.** Drop terms any paper in any field would 
    use with the same gloss — *functional unit*, *control group*, *sample 
    size*, *calibration curve*. Such a term grounds to one node every paper 
    attests and forms edges near the centroid that mediate nothing. 
    Generality alone is not the test: *compressive strength* and *scanning 
    electron microscopy* are broad but carry specific content and stay. 
 
 #### The complement pass — run once, over the finished draft 
 
 Draft the full term list first. Then re-read every gloss and drop any term 
 **that can only be glossed as the absence, remainder, or complement of 
 another term on the list.** *interfacial transition zone* glosses on its 
 own — a region of altered microstructure at the boundary between aggregate 
 and paste. *Bulk paste* glosses only as what is left over when that region 
 is removed. One of the pair carries the content; the other is its shadow. 
 Keep the first, drop the second. 
 
 This test is deliberately deferred. Applied while listing, it depends on 
 which term you happened to write down first and silently keeps whichever 
 half of the pair came earlier. Applied to the completed draft, it sees 
 both halves and you choose which one carries the content. 
 
 #### Fields 
 
 - **`id`** — `t01`, `t02`, … in extraction order. 
 - **`term`** — as written. 
 - **`senses`** — one or more. Most terms have one. 
 
 #### Senses 
 
 A term may carry more than one sense, and should when the paper genuinely 
 uses it at more than one level. 
 **Add a second sense only if it moves the word.** Two senses that say the 
 same thing at different verbosity are one sense; that is exclusion test 3 
 again. Two senses that read the same referent at different levels — the 
 instance and the mechanism — are complementary and both belong. 
 
 This matters most for clause-length terms. Gloss them **twice**: once at 
 the instance, once at the mechanism. The mechanism gloss must not name 
 another field — state the shape, and let the graph find who shares it. 
 
 - instance: "The change in which carbon dioxide rather than water governs 
   which minerals form." 
 - mechanism: "The displacement of one governing process by another when 
   conditions change." 
 
 Each sense carries: 
 
 - **`id`** — the term's id, suffixed: `t08.s1`, `t08.s2`, in order. 
   Senses are what the graph builds on; every reference elsewhere in this 
   output is to a sense id, never a bare term id. 
 - **`gloss`** — written by you. `GLOSS_TARGET` words, never below 
   `GLOSS_MIN` or above `GLOSS_MAX`. **A complete noun phrase with its 
   article, matching the lexicon it will be compared against:** 
   *"The replacement of calcium hydroxide with calcium carbonate triggered 
   by a chemical reaction."* 
   Not *"Replacement of calcium hydroxide…"*. No hedging, no citation, no 
   notation the definition doesn't need, no "the authors propose." Length 
   is two-sided — under `GLOSS_MIN` is as wrong as over `GLOSS_MAX`. 
 - **`definition`** — the paper's own defining sentence, verbatim, or null. 
   Null is normal and expected; do not paraphrase a passing mention into a 
   fake definition. 
 
 --- 
 
 ### RELATIONS 
 
 Any connection the paper states between two things. 
 
 **Both members must be sense ids you assigned.** `members` carries ids, 
 not prose; the paper's own phrasing goes in `paper_wording`, verbatim. If 
 an endpoint can't be expressed as an extracted sense, either the term was 
 missed — extract it — or the relation isn't between concepts and isn't 
 captured. 
 
 Naming senses rather than terms is what makes a relation buildable: edges 
 form between senses, and a polysemous endpoint given as a bare term leaves 
 the choice of sense to a downstream process that has less information than 
 you do. 
 
 This binds hardest on `explains`, whose endpoints in prose are 
 propositions. "Carbonate precipitation within pore spaces led to increased 
 apparent density" relates two events; extract the process clauses as terms 
 if they qualify, otherwise use the concepts underneath. A row whose 
 members are unextracted event descriptions produces no edge. 
 
 The two members may be **two senses of the same term** — `["t57.s1", 
 "t57.s2"]`. Two studies measuring one quantity, or two naming traditions 
 for one referent, are the substrate's native shape, not a degenerate case. 
 The two members may never be the identical sense id. 
 
 | type | fires on | 
 |---|---| 
 | `same_phenomenon` | the same thing under different names, or shown identical | 
 | `is_instance_of` | a specific thing fills a named general role — *CKD acts as an internal activator* | 
 | `extends` | one concept generalizes, specializes, or builds on another, same domain | 
 | `explains` | an observed effect attributed to a named cause, same domain | 
 | `refutes` | a claim, conjecture or bound shown false or unattainable | 
 | `contrasts_approach` | opposing solutions to the same problem | 
 | `analogous_mechanism` | a parallel across different substrates, **stated by the paper** | 
 | `shares_technique` | the same method on a different subject, including the paper reusing its own | 
 | `measures_same_quantity` | two methods estimate one quantity and the difference is reported | 
 | `stated_non_finding` | a connection the paper names and explicitly cannot explain | 
 
 `analogous_mechanism` fires on what the paper asserts, not on what you 
 notice. An analogy you supply is unattested and will be indistinguishable 
 downstream from one the authors defended. 
 
 --- 
 
 ### STRUCTURES 
 
 Where the paper builds a classification or draws a load-bearing 
 distinction. 
 
 A structure is one **axis** and two or more **poles**. Each pole carries a 
 short label and the sense ids that sit on that side. Members are sense 
 ids, under the same discipline as relations: a pole whose occupant is not 
 an extracted sense means the term was missed, and a structure with fewer 
 than two populated poles is not a distinction and is dropped. 
 
 **`axis` names the dimension along which the poles differ, not the topic 
 they belong to.** "Whether an optimum is reachable by continuous motion" 
 is an axis; "maxima" is a topic. If the axis stands without the paper's 
 subject matter, it will bridge. 
 
 **`pole.label`** names the side in the axis's own vocabulary — *volumetric* 
 / *geometric*, not a restatement of its members. 
 
 **`paper_wording`** — the sentence where the paper draws the split, 
 verbatim. Provenance for the axis, which is otherwise entirely your 
 writing. 
 
 --- 
 
 ### OUTPUT 
 
 JSON only. Every key present; absent values null, never omitted. 
 
 ```json 
 { 
   "terms": [{ 
     "id": "t21", "term": "γ-belite", 
     "senses": [{ 
       "id": "t21.s1", 
       "gloss": "The dicalcium silicate polymorph that does not react appreciably with water.", 
       "definition": "This indicates that γ-belite undergoes carbonation without the need for hydration" 
     }] 
   }, { 
     "id": "t08", "term": "shift from hydration-dominated to carbonation-dominated phase evolution", 
     "senses": [ 
       { "id": "t08.s1", 
         "gloss": "The change in which carbon dioxide rather than water governs which minerals form.", 
         "definition": null }, 
       { "id": "t08.s2", 
         "gloss": "The displacement of one governing process by another when conditions change.", 
         "definition": null } 
     ] 
   }], 
 
   "relations": [{ 
     "id": "r07", "members": ["t32.s1", "t45.s1"], "type": "explains", 
     "paper_wording": "The formation of this dense carbonate shell reduces pore connectivity and limits the diffusion of CO2 toward unreacted cores" 
   }, { 
     "id": "r15", "members": ["t57.s1", "t57.s2"], "type": "measures_same_quantity", 
     "paper_wording": "achieving compressive strengths exceeding 45 MPa after CO2 curing. This behavior is similar to that observed in the present study for series LF20-C, which reached compressive strengths above 48 MPa" 
   }], 
 
   "structures": [{ 
     "id": "s05", 
     "axis": "how much of a void space there is versus how it is shaped", 
     "poles": [ 
       { "label": "volumetric", "members": ["t12.s1", "t19.s1"] }, 
       { "label": "geometric",  "members": ["t23.s1", "t31.s1"] } 
     ], 
     "paper_wording": "Neither total porosity nor macro-porosity correlated with strength, whereas pore-specific surface area and tortuosity did" 
   }] 
 } 
 ```

