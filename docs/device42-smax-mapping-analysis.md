# Device42 - SMAX Data Structure Comparative Mapping and Analysis Report

## 1. General Structure Comparison

| Feature         | Device42 JSON                                 | SMAX JSON (entities/properties)         |
|-----------------|----------------------------------------------|-----------------------------------------|
| Root Field      | Devices (array)                              | entities (array)                        |
| Device Detail   | Each device is an object                     | Each entity is an object                |
| Sub-objects     | As array or object (e.g., ip_addresses)      | As JSON string (e.g., IpAddresses)      |
| Meta Info       | limit, total_count, offset                    | meta (completion_status, error, etc.)   |

---

## 2. Main Field Mappings

| Device42 Field      | SMAX Field (properties)      | Description/Mapping Note |
|---------------------|-----------------------------|--------------------------|
| name                | DisplayLabel                | Device name              |
| id / device_id      | Id                          | Unique device ID         |
| ram                 | MemorySize                  | RAM, in MB               |
| cpucount, cpucore   | Cpus (JSON string)          | CPU count/cores          |
| serial_no           | (Optional)                  | Serial number            |
| os                  | OsName                      | Operating system name    |
| osver, osverno      | (Optional)                  | OS version               |
| ip_addresses        | IpAddresses (JSON string)   | IP addresses             |
| hdd_details, hddsize| DiskDevices (JSON string)   | Disk information         |
| last_updated        | LastUpdateTime              | Last update time         |
| category            | (Optional)                  | Category                 |
| tags                | (Optional)                  | Tags                     |

---

## 3. Sub-objects and Array Fields

### IP Addresses
- **Device42:**  
  "ip_addresses": [ {"ip": "192.168.1.1", ...} ]
- **SMAX:**  
  "IpAddresses": "{\"IpAddress\":[{\"IpValue\":\"192.168.1.1\", ...}]}"
- **Mapping:**  
  Each ip_addresses element in Device42 should be converted to an IpAddress object in the SMAX IpAddresses JSON string.

### CPU Information
- **Device42:**  
  cpucount, cpucore, cpuspeed as separate fields.
- **SMAX:**  
  "Cpus": "{\"Cpu\":[{\"CpuClockSpeed\":\"2400\",\"CpuCoreNumber\":\"2\", ...}]}"
- **Mapping:**  
  CPU-related fields in Device42 should be converted to the JSON string format expected by SMAX.

### Disk Information
- **Device42:**  
  hdd_details (array), hddsize, hddcount fields.
- **SMAX:**  
  "DiskDevices": "{\"DiskDevice\":[{\"DiskSize\":\"100\",\"DiskModelName\":\"scsi\", ...}]}"
- **Mapping:**  
  Disk details in Device42 should be converted to the SMAX DiskDevices JSON string format.

---

## 4. Transformation and Considerations

- **JSON String Format:**  
  SMAX expects some fields (IpAddresses, Cpus, DiskDevices) as JSON strings. Arrays/objects from Device42 must be stringified before sending to SMAX.
- **Empty/Null Values:**  
  Some fields in Device42 can be null or empty string. When sending to SMAX, these fields should be left empty or assigned a default value.
- **Field Name Differences:**  
  Field names are not always identical; mapping is required.
- **Additional Fields:**  
  There may be fields in Device42 not present in SMAX (e.g., custom_fields, tags) or vice versa (e.g., DisplayLabel). Add or skip as needed.
- **Time Format:**  
  Device42 uses ISO datetime, SMAX uses epoch (milliseconds). Conversion is required.

---

## 5. Example Mapping Table

| Device42 Field         | SMAX Field (properties)      | Transformation Note         |
|------------------------|-----------------------------|----------------------------|
| name                   | DisplayLabel                | Direct                     |
| id / device_id         | Id                          | Direct                     |
| ram                    | MemorySize                  | Should be in MB            |
| cpucount, cpucore      | Cpus                        | Convert to JSON string      |
| serial_no              | (Optional)                  | Can be a custom field      |
| os                     | OsName                      | Direct                     |
| osver, osverno         | (Optional)                  | Can be a custom field      |
| ip_addresses           | IpAddresses                 | Convert to JSON string      |
| hdd_details, hddsize   | DiskDevices                 | Convert to JSON string      |
| last_updated           | LastUpdateTime              | Convert ISO to epoch        |
| category, tags         | (Optional)                  | Can be a custom field      |

---

## 6. Conclusion

- **Mapping and Transformation:**  
  Data from Device42 should be transformed to match the structure expected by SMAX. Especially, array/object fields must be stringified and time format converted.
- **Missing/Empty Fields:**  
  For missing or empty fields, assign default values or skip as per business rules.
- **Field Mapping:**  
  Main fields can be mapped easily, but some special fields may require additional mapping or custom field definitions.

---

### Note: RAM Mapping
- The `ram` field in Device42 should be mapped to `MemorySize` under properties in SMAX.
- Example:  
  Device42:  
  ```json
  "ram": 4096
  ```
  SMAX:  
  ```json
  "MemorySize": 4096
  ```

This mapping ensures RAM is correctly represented in SMAX.

For further mapping or example, please specify! 