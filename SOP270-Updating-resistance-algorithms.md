# SOP270 Rega resistance algorithm management

## Purpose

To provide a standard process for managing the implementation and deployment of Rega resistance interpretation algorithm changes.

## Scope

Applies to:

+  Updating the Rega algorithm narrative (DOC and PDF) and ASI-XML files
+  Documenting changes made and controlling versions in a systematic manner
+  Testing the algorithm changes
+  Deploying the algorithm to the central repository used by all RegaDB instances
+  Deploying the algorithm to the Rega online resistance interpretation website
+  Distributing the algorithm to other interpretation services that use it eg, GRADE and HIVDB
+  Publishing the new algorithm on the KUlLeuven - Rega Institute website

Does not apply to:

+  Creating the rules for the Rega algorithm
+  Updating the resistance report templates in the UZL or any other instance of RegaDB to display the interpretation made using the new rules
+  Finding and deploying new algorithms from other sources such as ANRS and HIVDB
+  Validating the clinical correctness / applicability of the algorithm rules

## Related SOPs

+  Updating the report templates (SOP not yet written)
+  Adding new algorithms from other sources (don't know if it exists)

## Policy

+  The algorithm documentation, this SOP, the narrative rules and the XML files are stored and managed using Github, in the Rega-ASI repository.
<br><br>
https://github.com/rega-cev/Rega-ASI
<br><br>
+  Rule changes required by the academic / clinical staff must be included in the documentation as requested. This should include the actual rule changes required, the rationale for those changes and any literature references that may apply. This is stored in the repository as rega-algorithm-rules-document.md. Previous text in the rules document can be overwritten with the new changes since versions are controlled using github.
+  Each change or group of related changes in the rules document must be captured as a change request ("issue") in Github. Team members decide how fine-grained issues must be (eg. all mutations for one drug in one issue, or all mutations for the entire class as one issue, etc.). This is decided depending on the type of changes requested (eg. addition of gag rules are seperate issues to the issues logged for PI changes, even if same drug)
+  New Rega algorithms must be tested before deployment and distribution to ensure the XML can be interpreted by at least, the Stanford and RegaDB interpretation tools. 
+  The Rega algorithm must be sent to HIV-GRADE and Stanford HIVDB when finalised.
+  The Rega algorithm must be updated on the Rega resistance interpretation website when finalised. 
+  The Rega algorithm must be published on the KULeuven - Rega Institute website when completed.
+  All algorithms must be tested on an instance of RegaDB of same configuration as the UZL instance to ensure the production system will not be affected when the algorithm is deployed.

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

> this section probably needs a bit of work, or can be skipped entirely??

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
+  Mutations listed at the same position (eg. 101E and 101P) that have different scores are encoded in the XML as a MAX statement. If the mutations have the same scores then they are simply combined (eg. 106ED > E or D at position 106)
+  Maraviroc is not interpreted (can be interpreted using the geno2pheno tool if required).

## Procedure
### Managing new changes

1.  From the rule changes document received from the clinical virology team (email, MS Word etc), replace the text in the rega-algorithm-rules-document.md for the particular drug. In case of drugs where no changes were requested, replace the existing text (changes requested in previous versions) in the rega-algorithm-rules-document.md with "no changes required" or similar to indicate that the new version will not involve any changes to that particular drug.
1.  From rega-algorithm-rules-document.md, rewrite the changes as a series of issues in github.
	1.  **Title**: "{drug} rule changes", or a more specific message if required
	2.  **Comment**: Copy and paste all the changes that are to be included in this issue from rega-algorithm-rules-document.md. What is included will depend on how fine-grained the changes made will be between tests (the entire team should decide this). In the majority of cases, all the change required for the particular drug can be added to the issue but in some cases it may be neccesary to split the changes required across two issues (eg. the gag - PI changes mentioned under Policy)
	3.  **Assign** the issue to the person responsible for implementing the changes to both narrative and XML files.
1.  Clone or pull the repository locally

### Changing the algorithm
1.  Changes are added on a new branch.
1.  Address one issue at a time but in both the narrative and XML files.
  1. Make the same change in both files so that they correspond at all times.
  1. Some changes are required in only one of the files (eg. author affiliations occur in the narrative file only).
  1. Check the quality of the edits

### Notes about drug abbreviations
+  Choose drug abbreviations that are the same as Stanford. If this is not possible because in regaDB we want a specific abbreviation that may be different from Stanford, then in the algorithm use the Stanford abbreviation and let Ewout change the regaDB code to map from the Stanford abbreviation used in the rega algorithm to the abbreviation used in regaDB. DRV/c and TAF were handled this way in v10: the algorithm was made with DRV/r and TDF only but in regaDB we also have TAF and DRV/c as seperate drugs so the mapping was made so that the interpretation of TAF is done according to the TDF rules. 

### Test the XML
1.  Test the algorithm using the Stanford tool as a minimum (more tests in the Additional tests section).
  1.  Navigate to http://sierra2.stanford.edu/sierra/servlet/JSierra?action=algMutationsInput
  1.  Under the Algorithms section in section B, upload the changed XML.
  1.  Enter mutations from the XML into the mutation text boxes, or use the combo boxes. Be sure to use the correct drug class.
1.  If any problems are found, fix and retest. If the XML syntax is incorrect, a blank screen is typically displayed.

### Commit and push
1.  Once the algorithm appears to be working correctly:
  1.  Commit the changes (both files, one commit) and reference the associated issue in the commit.
  2.  Push changes
  1.  Refresh issue page and close the associated issue on Github and commment appropriately.

### Additional testing (as required / possible)
#### Other Stanford tools
#### GRADE tools
#### Kristof / Dutch discordance tool (?)
#### Jens testing tool (?)
#### RegaDB compiler / Sequence tool
1.  Move the new algorithm xml and a fasta file to crunchie
1.  Run ./asiprocess myAlgorithm.xml mySequence.fasta
1.  Check the output

#### RegaDB development version (manual add exports)

1.  Implement the algorithm in a development or test version of RegaDB and check that it works
  1.  Upload the algorithm manually through the interface of a development / test version of RegaDB, preferably with a large dataset
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
      1. file content: click browse to select the algorithm from your computer
      1. click upload
      1. click add
  1. Click ok
  1. Restart RegaDB
  1.  Add a test sample or edit an existing sample and save it.
  1.  Check that the resistance is interpreted correctly
  1.  Batch check:
    1.  Export resistance data ("before" dataset)
    1.  Perform a batch analysis
	    1. Administrator >> batch test >> Add
		1. In the dropdown, select the new algorithm >> OK
		1. The progress indicator will show when the batch analysis is done, then click OK
		1. check a few patients to see if the results are ok.
    1.  Export the resistance data again ("after" dataset)
    1.  Diff the before and after datasets and check the changes to verify the software uses the algorithm correctly

#### RegaDB: Test deployment on test system from the central repository
1. On the test system, delete any results associated with the new algorithm added during previous testing
  1. Login to Postgres: psql -U regadb_user regadb_password
  1. Find the index for the algorithm:
  >SELECT * FROM regadbschema.test;

  1. Delete all analysis results for this algorithm:
  > DELETE FROM regadbschema.test_result WHERE test_ii = <<index found before>>

  1. Remove the algorithm itself
  1. Administrator >> Test Settings >> Tests >> "algorihtm_X"
  1. Delete and confirm
1.  Add the algorithm to a test instance of the central repository that the test instance of RegaDB points to.
  1. Update from central repository
    1. On simulation, scroll down to Tests and check that the new algorithm displays as a new test. It should also have an associated new analysis entry ($WTS_URL)
  1. Run the update
  1. Once update from central is done check one patient to see if it works (as explained above)
  1. If one patient is good, perform a batch analysis and check (as explained above)

### Finalise
Once all issues have been addressed and testing is complete, the algorithm can be finalised.

1.  Decide on new version number (see “standards” section)
1.  Change ALGNAME and ALGVERSION tags in the XML
1.  Update history comment block in the XML and commit
1.  Rename the file and commit (git move)
1.  Make a release branch using the name of the version as the branch name
1.  Tag the commit as a release also
1.  Push to github repository, (with tags!), all branches

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
<br>


*NOTE: Let users know how long it will take and they shouldn't add any sequences. It takes approximately 1.1 seconds per viral isolate.*
<br>
  1.  Administrator >> batch test >> Add
  1.  In the dropdown, select the newly added algorithm
  1.  Ok
  1.  when done, click OK
  1.  Let users know when it's done
  1.  Update the resistance template (see other SOP)
