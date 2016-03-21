# SOP270 Rega resistance algorithm management

## Purpose

To provide a standard process for managing and deploying Rega resistance interpretation algorithm changes

## Scope

Applies to:

+  Updating the Rega algorithm narrative (DOC and PDF) and associated ASI-XML files
+  Documenting changes made and controlling versions in a systematic manner
+  Testing / validating the new algorithm for clinical and technical correctness
+  Deploying the algorithm to all instances of RegaDB
+  Distributing the algorithm to other interpretation services eg, GRADE and HIVDB
+  Publishing the new algorithm on the Rega website

Does not apply to:

+  Creating the rules for the Rega algorithm
+  Updating the resistance report templates in the UZL instance of RegaDB to display the interpretation made using the new rules
+  Finding and deploying new algorithms from other sources notably, ANRS and HIVDB

## Related SOPs

+  Updating the report templates (SOP not yet written)
+  Adding new algorithms from other sources

## Policy

+  The algorithm documentation, this SOP, the narrative and XML files are stored and managed using Github, using the Rega-ASI repository.
<br><br>
https://github.com/rega-cev/Rega-ASI
<br><br>
+  Rule changes required by the academic / clinical staff must be included in the documentation. This includes the rule changes required, the rationale for those changes and any literature references that may apply.
+  Each change or group of related changes in the rules document must be captured as a change request ("issue") in Github. Team members decide how fine-grained issues must be (eg. all mutations for one drug in one issue, or all mutations for the entire class as one issue, etc.)
+  New Rega algorithms must be tested / validated before deployment and distribution to verify both technical and scientific/clinical soundness.
+  The Rega algorithm must be sent to HIV-GRADE and Stanford HIVDB when finalised.
+  The Rega algorithm must be published on the Rega website when completed.
+  All algorithms must be tested on the live UZL instance to make sure the algorithm doesn't cause a failure in the software because of configuration differences
+  After deployment the configuration file and resistance report of the UZL instance of RegaDB must be updated to display the new algorithm by default (the resistance report update process is a separate SOP).

## Standards
### Versioning standard

+  The Rega algorithm name and version must comply with the pattern REGA vX.Y.Z, where:
  +  X is incremented by 1 when:
      +  a new virus type, drug class and/or drug is included in the rules
      +  the GSS for a drug is changed
      +  an updated ASI standard, that requires changes to the structure of the file, is implemented
  + Y is incremented by 1 when x is not incremented and:
      +  Rules / mutation lists for drugs already in the algorithm are updated.
  +  Z is increment by 1 when x and y are not incremented and:
      +  Errors in the narrative rules or programming of the XML (ie bugs) are fixed

### XML Standards

> this section probably needs a bit of work, or can be skipped entirely

+  The XML file is used by the resistance interpretation software to determine how to increment a resistance score based on the type and quantity of mutations in the input sequence. Some mutations cause higher level resistance than others and are scored higher.
+  Each mutation or mutation combination is given a weight. The software compares the mutation list in the algorithm XML against the mutation list from the sequence and assigns a total score. The algorithm contains a score range which uses the total score and assigns an order value. The order value varies from 1 to 6 and represents the resistance interpretation, as follows:

| Order | SIR |GSS  |
| :---- | :-- | :-- |
| 1     | S   |1.5  |
| 2     | S   |1.0  |
| 3     | S   |0.75 |
| 4     | I   |0.5  |
| 5     | I   |0.25 |
| 6     | R   |0.0  |

+  The score range in the XML corresponds to the “score at least” statement which each drug listing starts with in narrative file.
+  Mutations listed at the same position in the PDF (ex. 101E and 101P) that have different scores are encoded in the XML as a MAX statement. If the mutations have the same scores then they are simply combined (ex. 106ED > E or D at position 106)
+  Maraviroc isn’t in the XML because interpretation relies on the geno2pheno tool.

Basic structure of XML using RPV as example:

> insert example XML here

## Procedure
### Changing the algorithm

Note: Clinical virologists and/or clinicians determine what changes to the rules are needed and should be implemented and write this information in the rules document (outside scope of this SOP).

1.  Add the **rules document** as a new version to **Github**. Make the updates to each drug a seperate commit. 
1.  From the rules document, rewrite the changes as a series of **issues** in github. 
	1.  **Title**: "{drug} rule changes", or a more specific message
	2.  **Comment**: Copy paste the changes that will are to be included for this issue from the rules document. What is included will depend on how fine-grained the issues should be (the team decides this) or what was decided should be included (some things may be left out until later).
	3.  **Assign** it.
1.  Clone / **pull** the repository locally
1.  Address one issue at a time in both the narrative and XML files, writing the changes required into both files.
  1. Make the same change in both files so that they correspond at all times.
  1. Some changes are required in one of the files only, eg. author affiliations occur in the narrative file only.
  2. Check the quality of the edits
1.  Commit the changes (both files, one commit). Reference the associated issue in the commit.
1.  Test the algorithm using the Stanford tool (see below)
1.  If any problems are found, fix, retest and amend previous commit.
1.  When the algorithm appears to be working correctly, update the issue status to “fixed” on Github
1.  Do other Tests
1.  Finalise (see below)

### Testing
#### Kristof /Dutch discordance tool (?)
#### Jens testing tool (?)
#### Sequence tool

Run the fasta file against the current XML, then make the changes and run it again using the new XML. Diff the results.

#### RegaDB dev version (manual add and exports)

1.  Implement the algorithm in a development or test version of RegaDB with a large dataset and check that it works as expected
  1.  Upload the algorithm manually through the interface of a development or test version of RegaDB with a large dataset
    1. Administrator >> test settings >> test >> Add
      1. test: REGA vx.y.z
      1. test type: Genotypic Susceptibility Score (GSS) (HIV-1)
      1. analysis: check
    1. Under analysis group enter/select data on the form as follows (exactly)
      1. type: wts
      1. URL: $WTS_URL
      1. service name: regadb-hiv-resist
      1. account: public
      1. password: public
    1. Click refresh
      1. base_inputfile: viral_isolate
      1. base Outputfile: interpretation
    1. Under analysis data group
      1. name: asi_rules (where the git repository is)
      1. click browse to select the algorithm from your computer
      1. click upload
      1. click add
  1. Click ok
  1. Restart RegaDB
  1.  Add a test sample and save it.
  1.  Check that the resistance is interpreted correctly
    1.  Export resistance data ("before" dataset)
    1.  Perform a batch analysis
    1.  Export the resistance data again ("after" dataset)
    1.  Diff the before and after datasets and check the changes to verify the software uses the algorithm correctly (differences are what was expected)


####RegaDB: Test deployment on test system from the central repository
1. The algorithm needs to be added to the central repository by Ewout
1. On the dev system, delete any results associated with the new algorithm added during previous testing
  1. Login to Postgres: psql -U regadb_user regadb_password
  1. Find the index for the algorithm: SELECT * FROM regadbschema.test;
  1. Delete all analysis results for this algorithm: DELETE FROM regadbschema.test_result WHERE test_ii = <<index found before>>
  1. Remove the algorithm itself
  1. Administrator >> Test Settings >> Tests >> <<name of the algorithm>>
  1. Delete and confirm
  1. Update from central repository
    1. On simulation, scroll down to Tests and check that the new algorithm displays as a new test. It should also have an associated new analysis entry ($WTS_URL)
  1. Run the update
  1. Once update from central is done check one patient to see if it works
  1. If one patient is good, perform a batch analysis
    1. Administrator >> batch test >> Add
    1. In the dropdown, select the new algorithm >> OK
    1. The progress indicator will show when the batch analysis is done, then click OK
    1. check a few patients to see if the results are ok.

### Finalise
Once all issues have been addressed and testing is complete, the algorithm can be finalised.
1.  Decide on new version number (see “standards” section)
1.  Change ALGNAME and ALGVERSION tags in the XML
1.  Update history comment block in the XML
1.  Commit
1.  Rename the file and commit (git move)
1.  Tag the commit as a release
1.  Push to github repository, (with tags!)

### Distribution
#### Rega website
1.  Print the narrative file as a PDF (Rega_HIV1_Rules_vx.y.z.pdf)
1.  Make a temporary copy of the XML and rename it RegaHIV1Vx.y.z.xml
1.  Go to Rega website
<br><br>
https://rega.kuleuven.be/cev/avd/software/software/rega-algorithm
<br><br>
  1.  Login
  1.  Copy-paste the text, pdf icon and xml icon from the most recent row in the table into a new row above
  1.  Change the date to the current date
  1.  Adjust the author list if needed
  1.  Edit the link of the xml icon upload the new XML file
  1.  Upload the PDF and link to it in the same way
  1.  Change the version text
  1.  Review changes and check the links work correctly
  1.  Download the files and compare (diff) them to ensure the correct files were uploaded
  1.  Edit the links and text on the page https://rega.kuleuven.be/cev/avd/software
> the rega website can be made to point to the github repository - that means we don't have to update the website also

#### Distribution to partners
1.  Generate a test dataset from HIVDB
<br><br>
http://sierra2.stanford.edu/sierra/servlet/JSierra?action=algSequenceInput
<br><br>
1.  Send the XML along with the test dataset to HIV GRADE (obermeier@bergdoctor.de)
    1.  Test dataset here:
    <br><br>
    \\toaster\home13\fferre0\Work\ResistanceAlgorithms\Testing\Testing2014\test-dataset
    <br><br>
    1.  To make a test dataset:
        1.  Stanford algorithm tool
        1.  Tick only Rega algorithm
        1.  Upload first fasta
        1.  Select spreadsheet as output
        1.  Click analyse
        1.  Download and extract results
        1.  Repeat process with other fasta files
        1.  ZIP results and fasta files
        1.  Commit the zip file with message 'dataset and results for martin obermeyer after Rega algorithm vx.y.z deployment on GRADE '
        1.  Send the PDF, XML and dataset (with results) to Martin Obermeyer at the HIV-GRADE project website
1.  Send the algorithm to HIVDB (syrhee@stanford.edu)

### Deployment in GHB
1.  Add the algorithm to the central repository (Ewout)
1.  Update from central
1.  Run the RegaDB exports test again
1.  Update the resistance report template, test it and deploy it (other SOP)
1.  Perform the batch analysis
<br><br>
NOTE: Let users know how long it will take and they shouldn't add any sequences. It takes approximately 1.1 seconds per viral isolate.
<br><br>
  1.  Administrator >> batch test >> Add
  1.  In the dropdown, select the newly added algorithm
  1.  Ok
  1.  when done, click OK
  1.  Let users know when it's done
  1.  Update the resistance template (see other SOP)
