# Terraform secrets

Add a secret with: `echo "<secret>" | gcloud secrets create <secret-name> --data-file=-`

Terraform secrets code should have depends_on defined for its step,
such that the its dependancy graph forms a line.

