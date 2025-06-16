# Device42 to SMAX Integration Project

This project automates and securely transfers server (device) data from the Device42 inventory management system to the SMAX (Service Management Automation X) platform. The structure works in two modular stages on Ansible AWX.

---

## Directory Structure

```
.
├── playbooks/
│   ├── fetch-device42-devices.yml      # Playbook to fetch data from Device42 and save to file
│   └── send-to-smax.yml               # Playbook to send data to SMAX and obtain token
├── docs/
│   ├── device42-json-field-analysis.md         # Device42 JSON field analysis
│   ├── device42-smax-mapping-analysis.md       # Device42-SMAX field mapping and transformation analysis
│   └── smax-response-field-analysis.md         # SMAX response JSON field analysis
└── README.md
```

---

## Workflow and Usage

### 1. Fetch Devices from Device42
- **Playbook:** `playbooks/fetch-device42-devices.yml`
- **Purpose:** Fetches devices from the Device42 API and saves them to the `device42_devices.json` file.
- **Input (AWX Extra Vars):**
  - `device42_api_url`: Device42 API endpoint
  - `device42_auth_header`: Authorization header for Device42 (e.g., Basic ...)
  - `chunk_size`: (Optional) Number of devices to process per batch (default: 1)

### 2. Send Devices to SMAX
- **Playbook:** `playbooks/send-to-smax.yml`
- **Purpose:** Reads the devices saved in the previous step, dynamically obtains a token from SMAX, and sends the devices in the required format to SMAX.
- **Input (AWX Extra Vars):**
  - `smax_token_url`: SMAX token endpoint
  - `smax_user`: SMAX username
  - `smax_pass`: SMAX password
  - `smax_api_url`: SMAX bulk insert endpoint
  - `chunk_size`: (Optional) Number of devices to send per batch (default: 1)

### 3. File Flow
- When the first playbook is run, devices are saved to the `device42_devices.json` file.
- The second playbook reads this file and transfers the data to SMAX.
- Both playbooks are run separately as AWX Job Templates.

---

## Field Mapping and Transformation
- For field mapping and transformation rules between Device42 and SMAX data models, see `docs/device42-smax-mapping-analysis.md`.
- For Device42 JSON structure and field descriptions, see `docs/device42-json-field-analysis.md`.
- For SMAX response structure and field descriptions, see `docs/smax-response-field-analysis.md`.

---

## Additional Notes
- Playbooks are designed to be fully parameterized and dynamic. Credentials and chunk_size can be provided via the AWX interface.
- JSON transformations and field mappings are implemented to match SMAX requirements.
- The project is modular for easy maintenance and extensibility.

---

## References and Detailed Analyses
- [Device42 JSON Field Analysis](docs/device42-json-field-analysis.md)
- [Device42-SMAX Field Mapping Analysis](docs/device42-smax-mapping-analysis.md)
- [SMAX Response JSON Analysis](docs/smax-response-field-analysis.md)

---

For any questions or further development, please refer to the analysis files in the docs folder or contact the project maintainers. 