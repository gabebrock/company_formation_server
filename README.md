# Company Formation Server

This server accepts company formation data and generates Articles of Incorporation for Delaware corporations.

## Setup

1. Install the required dependencies:
```bash
pip install -r requirements.txt
```

2. Run the server:
```bash
python app.py
```

## API Usage

Send a POST request to `/form-company` with a JSON payload in the following format:

**For Delaware and California formation**
```json
{
    "company_name": "Acme Corp, Inc.",
    "state_of_formation": "DE",
    "company_type": "corporation",
    "incorporator_name": "John Smith"
}
```

**For New York formation**
```json
{
    "company_name": "Acme Corp, Inc.",
    "state_of_formation": "NY",
    "company_type": "corporation",
    "incorporator_name": "John Smith",
    "county": "QUEENS COUNTY", # Required for NY formations
    "address": "456 Park Ave, NY 11101" # Required for NY formations
}
```

### Validation Rules:
- Company name: Can contain alphanumeric characters, spaces, commas, periods, apostrophes, and ampersands
- State of formation: Must be a valid US state or territory code
- Company type: Must be either "corporation" or "LLC"
- Incorporator name: Required field
- County: Required for New York formations
- Address: Required for New York formations

### Response:
- For Delaware corporations: Returns a PDF file containing the Articles of Incorporation
- For New York corporations: Returns a JSON object containing the Articles of Incorporation as a base64 encoded PDF, a warning if any required fields were missing, and the autofilled values for those fields
- For other states/company types: Returns a 400 error indicating unsupported formation type

If you submit a `curl` request to the `/form-company` endpoint, you will receive a JSON response containing the Articles of Incorporation as a base64 encoded PDF, a warning if any required fields were missing, and the autofilled values for those fields.

If you submit a `curl` request to the `/form-company` endpoint with a missing required field, you will receive a JSON response containing a warning message indicating which required fields were missing and the pdf as a base64 encoded string instead of a pdf until you fill out all the required fields. Add this additional command to your incomplete curl to save the pdf to a file: `| jq -r '.pdf_base64' | base64 -d > output.pdf`.

```bash
#Example: Missing county and address
curl -X POST http://localhost:8080/form-company \
  -H "Content-Type: application/json" \
  -d '{"company_name": "NY Corp", "state_of_formation": "NY", 
       "company_type": "corporation", "incorporator_name": "Jane"}'

# Example response
{
  "warning": "You forgot to fill out county, address. We helped you out, but the IRS may be coming after you.",
  "pdf_base64": "JVBERi0xLjMKJcTl...",
  "filename": "NY Corp_certificate.pdf",
  "missing_fields": ["county", "address"],
  "autofilled_values": {
    "county": "NEW YORK COUNTY",
    "address": "20 W 34th St., New York, NY 10001"
  }
}
```

If you submit a `curl` request to the `/form-company` endpoint with an invalid state of formation, you will receive a JSON response containing a 400 error indicating unsupported formation type as only CA, DE, and NY are supported.

If you submit a `curl` request to the `/form-company` endpoint with an invalid company type, you will receive a JSON response containing a 400 error indicating unsupported company type as only corporation and LLC are supported.



## Example cURL Request:
```bash
curl -X POST http://localhost:5000/form-company \
  -H "Content-Type: application/json" \
  -d '{
    "company_name": "Acme Corp, Inc.",
    "state_of_formation": "DE",
    "company_type": "corporation",
    "incorporator_name": "John Smith"
  }'
```
