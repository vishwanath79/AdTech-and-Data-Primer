# Privacy by Design in Action (Patterns for Engineers)

Here are some design patterns that show how Privacy by Design works:

### **Minimizing Data**

**Rule:** Only gather what you need.  Don't keep things you don't need.

**For example:**
- Instead of storing plaintext emails, store hashed ones. - Before storing data, combine it (no raw events).
- If you only need the country, don't collect IP addresses. - Delete them after 90 days if you don't need them for business. -level location
- Minimize or eliminate PII relentlessly

**Code example (bad):**
```python
# Storing everything "just in case"
user_event = {
    "user_id": "12345",
    "email": "user@example.com",
    "ip_address": "192.168.1.1",
    "user_agent": "Mozilla/5.0...",
    "location": {"lat": 47.6062, "lon": -122.3321},
    "timestamp": "2025-01-15T17:32:11Z",
    "page": "/products/guitar",
    "referrer": "https://google.com/search?q=guitar",
    "session_duration": 342,
    "previous_pages": ["/", "/products", "/products/guitar"]
}
```

**Code example (better):**
```python
# Store only what you need for analytics
page_view = {
    "anonymous_id": hash(user_id),  # One-way hash, can't reverse
    "country": "US",  # No precise location
    "device_type": "mobile",  # Simplified user agent
    "page": "/products/guitar",
    "referrer_domain": "google.com",  # Drop query params
    "timestamp": "2025-01-15T17:00:00Z"  # Rounded to hour
}
```

You can still do analytics. You just can't personally identify or track individuals.

### **Pseudonymization**

**Principle:** Replace identifying information with pseudonyms.

**For example:**
- Hash email addresses before saving them - Hashing has some drawbacks when sharing data with other people (for example, they might have a bigger database and be able to reverse the hash or match it to the original email address)
- Use random UUIDs instead of IDs that go in order
- Separate behavioral data from PII

**Architecture:**
```
User Service (stores PII)
user_id: "uuid-abc-123" └── email: "zakk@example.com"
    Name: "Zakk Mild" Address: "123 Rock St"

Analytics Service (keeps track of behavior)
└── anonymous_id: "hash-xyz-789"   #  Can't be undone
    ├── last_visit: "2025-01-15" ─── page_views: 47
    └── favorite_category: "guitar"
```

Hackers can get behavioral data from the analytics database, but they can't link it to real people.

### **Data Encryption**

**Principle:** Protect data at rest and in transit.

**In practice:**
- HTTPS for all traffic (not just login pages)
- Database encryption (PostgreSQL, MySQL and other databases support this)
- Encrypted backups
- Application-level encryption for sensitive fields

**Example:**
```python
# Encrypting sensitive data before storage
from cryptography.fernet import Fernet

key = Fernet.generate_key()
cipher = Fernet(key)

# Store encrypted
email_encrypted = cipher.encrypt(b"zakk@example.com")
db.store(user_id, email_encrypted)

# Retrieve and decrypt only when needed
email_encrypted = db.retrieve(user_id)
email = cipher.decrypt(email_encrypted)
```

### **Access Controls**

**Principle:** Only certain people should be able to see user data.

**In practice:**
- Role-based access control (RBAC) - Principle of least privilege (give the least amount of access needed)
- Audit logs (keep track of who accessed what)
- Protect sensitive data with more than one form of authentication

**Example:**
- **Engineers:** Can look at anonymized analytics but not PII - **Support team:** Can see user emails and order history (with audit logging)
- **Team in charge of money:**  Can see payment information but not browsing history

### **Policies for Keeping Data**

**Rule:** Don't keep data forever.  If you don't need it anymore, delete it.

**In real life:**
- Set retention periods for each type of data - Set up automated deletion jobs - Follow legal hold rules (don't delete data if it's being investigated)

**Policy example:**
- **User account data:** Kept while the account is active and deleted 90 days after it is closed. - **Analytics data:** Kept for 13 months, then deleted along with the raw data. - **Customer support tickets:** Kept for 3 years, then archived. - **Financial records:** Kept for 7 years (a legal requirement).

**Implementation:**
```sql
-- Automated cleanup job
DELETE FROM events 
WHERE timestamp < NOW() - INTERVAL '13 months';

DELETE FROM user_sessions 
WHERE last_activity < NOW() - INTERVAL '90 days';
```

### **Right to Access (Data Export)**

**Principle:** Users can request all data you have about them.

**In practice:**
- Build data export tools
- Package data in readable format (JSON, CSV)
- Respond within legal timeframe (30 days GDPR, 45 days CCPA)

**Example implementation:**
```python
def export_user_data(user_id):
    """Generate complete data export for user"""
    data = {
        "profile": get_profile(user_id),
        "purchase_history": get_purchases(user_id),
        "browsing_history": get_page_views(user_id),
        "preferences": get_settings(user_id),
        "consent_records": get_consent_history(user_id)
    }
    
    # Create downloadable file
    filename = f"user_data_{user_id}.json"
    with open(filename, 'w') as f:
        json.dump(data, f, indent=2)
    
    # Send download link to user
    send_email(user_email, download_link)
```

### **Right to Deletion (Right to Be Forgotten)**

**Rule:** Users can ask for their data to be deleted.

**In real life:**
- Find all the places where data is stored (backups, databases, logs, caches) - Create workflows for deleting data
- Use pseudonymization instead of hard deletion for legal or audit reasons
- Tell third parties you shared data with that you deleted it

**Example implementation:**
```python
def delete_user_data(user_id):
    """Delete all user data across systems"""
    
    # Delete from primary databases
    users_db.delete(user_id)
    analytics_db.delete_where(user_id=user_id)
    
    # Purge from caches
    redis.delete(f"user:{user_id}")
    
    # Anonymize in logs (can't delete logs, but remove PII)
    logs_db.update(
        {"user_id": user_id},
        {"user_id": "deleted_user", "email": "[deleted]"}
    )
    
    # Notify third parties
    notify_deletion_to_partners(user_id)
    
    # Log deletion for audit trail
    audit_log.record(f"Deleted user {user_id} per GDPR request")
```

### **Consent Management**

**Principle:** Track what users consented to, enforce those choices.

**In practice:**
- Store granular consent records (not just yes/no)
- Check consent before processing data
- Update consent when laws or policies change

**Example schema:**
```python
consent_record = {
    "user_id": "12345",
    "timestamp": "2025-01-15T10:30:00Z",
    "version": "2.1",  # Privacy policy version
    "consents": {
        "analytics": True,
        "marketing_emails": False,
        "personalized_ads": True,
        "third_party_sharing": False
    },
    "ip_address": "192.168.1.1",  # For audit trail
    "consent_method": "explicit_checkbox"
}
```

**Enforcement:**
```python
def can_process_for_ads(user_id):
    consent = get_latest_consent(user_id)
    return consent.get("personalized_ads", False)

# Before showing personalized ad
if can_process_for_ads(user_id):
    show_targeted_ad(user_id)
else:
    show_contextual_ad()  # No tracking required
```

## Privacy Impact Assessments (PIAs)

A **Privacy Impact Assessment** is a way to find and lower privacy risks before a feature goes live.

**When to do a PIA:**
- A new product or feature that handles personal data - Changes to how data is processed - The use of new technology (AI, biometrics, etc.)
- Whenever you try something new or risky

**Template for PIA:**

1. **What information will be gathered?** (Write down everything)
2. **Why is it needed?** (Explain each data point)
3. **Who will be able to get in?** (Teams inside the company and outside of it)
4. **How will it be protected?** (Encryption and access controls)
5. How long will it be kept?** (Time to keep)
6. **What are the dangers?** (Violations, abuse, and bias)
7. **How will you lower the risks?** (Specific controls)
8. **What rights do users have?** (Get in, delete, or opt-out)
9. What is the legal basis?**  (Consent, contract, and legitimate interest)
10. **Processors from other companies?**  (Vendors, sharing data)

**Output:** Write down the decisions, get the privacy/legal team to agree, and put the controls in place.

GDPR says that high-risk processing needs PIAs (also known as DPIAs, or Data Protection Impact Assessments).  Good practice even if the law doesn't require it.
