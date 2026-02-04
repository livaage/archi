# Indico Scraper Setup & Testing Guide

## Quick Start (5 minutes)

### 1. Install Dependencies

```bash
cd /Users/liv/archi
pip install -e .
```

This installs `markitdown[pdf,pptx]` and other requirements.

### 2. Run Quick Test

```bash
# Test with a public event (no credentials needed)
python test_indico_scraper.py
```

This will:
- Initialize the Indico scraper
- Fetch metadata from a sample public event
- Download and convert any slides to markdown
- Show you what was collected

**Expected output:**
```
Testing Indico Scraper
...
✓ Collected N resources
Resource breakdown:
1. event: Event Title...
2. contribution: Talk Title...
3. material: Slides converted to markdown...
```

---

## Finding Indico Events to Test

1. Go to https://indico.cern.ch/
2. Browse categories or search for events
3. Copy the event URL (e.g., `https://indico.cern.ch/event/1443798/`)
4. Add `indico-` prefix: `indico-https://indico.cern.ch/event/1443798/`

**Public vs Protected Events:**
- **Public events**: Work without credentials
- **Protected events**: Require CERN SSO login (see Step 3)

---

## 3. Set Up Credentials (For Protected Events Only)

If you need to scrape protected CERN Indico events:

### Option A: Environment Variables

```bash
export SSO_USERNAME='your-cern-username'
export SSO_PASSWORD='your-cern-password'
```

### Option B: Secrets File

Create `~/.a2rchi/.env` or add to your deployment's secrets:

```bash
SSO_USERNAME=your-cern-username
SSO_PASSWORD=your-cern-password
```

### Option C: Deployment-Specific Secrets

When using `a2rchi create`, place secrets in:
```
~/.a2rchi/a2rchi-<deployment-name>/secrets.env
```

---

## 4. Full Deployment Test

### Create Deployment Config

Edit `test_indico_config.yaml` (already created) or use an example:

```yaml
data_manager:
  sources:
    indico:
      enabled: true
      base_url: https://indico.cern.ch
      use_sso: false  # Set true for protected events
      slide_conversion:
        enabled: true
        formats: [pdf, pptx, ppt, odp]
```

### Create Weblist

Edit `test_indico.list` (already created) with your events:

```
indico-https://indico.cern.ch/event/YOUR_EVENT_ID/
indico-https://indico.cern.ch/category/YOUR_CATEGORY_ID/
```

### Run Data Manager

```bash
# Using CLI (if you have a full deployment)
a2rchi create test-indico --sources indico

# Or run data manager directly
python -c "
from src.data_manager.data_manager import DataManager
dm = DataManager(config_path='test_indico_config.yaml')
dm.run_ingestion()
"
```

---

## 5. What Gets Scraped

For each event URL, the scraper collects:

### Event Metadata
- Title, description, dates, location
- Saved as: `data/indico/event_<id>.md`

### Contribution (Talk) Metadata
- Speaker, title, abstract, time slot
- Saved as: `data/indico/contribution_<id>.md`

### Materials (Slides)
- Downloads PDF/PPTX/PPT/ODP files
- **Converts to markdown** using MarkItDown
- **Original files NOT stored** (markdown only)
- Saved as: `data/indico/<filename>.md`

### Metadata Preserved
Each resource includes:
```yaml
source_type: indico
event_id: "123456"
contribution_id: "789"
speaker: "Jane Doe"
original_filename: "slides.pdf"
original_format: "pdf"
converted_to_markdown: "true"
```

---

## 6. Troubleshooting

### "No module named markitdown"
```bash
pip install 'markitdown[pdf,pptx]>=0.1.0'
```

### "SSO authentication failed"
- Check `SSO_USERNAME` and `SSO_PASSWORD` are set correctly
- Verify you have valid CERN credentials
- Try with `use_sso: false` and a public event first

### "Event not found" or 403 errors
- Ensure the event ID is correct
- Check if event is public or requires authentication
- Enable SSO if needed: `use_sso: true`

### "Conversion failed"
- Check the file format is supported (pdf, pptx, ppt, odp)
- Some PDFs may be scanned images (no text to extract)
- Check logs for specific conversion errors

### No slides found
- Event may not have uploaded materials
- Check the event page manually to verify slides exist
- Some events only have linked materials (external URLs)

---

## 7. Configuration Options

### Basic Configuration

```yaml
data_manager:
  sources:
    indico:
      enabled: true
      base_url: https://indico.cern.ch
      use_sso: false
```

### Full Configuration

```yaml
data_manager:
  sources:
    indico:
      enabled: true
      base_url: https://indico.cern.ch
      
      # Authentication
      use_sso: true  # Enable for protected events
      sso_kwargs:
        headless: true  # Run browser in headless mode
        site_type: generic
      
      # Slide conversion
      slide_conversion:
        enabled: true
        formats:
          - pdf
          - pptx
          - ppt
          - odp
        store_originals: false  # Only markdown stored
        
        # Future: LLM-based image descriptions
        llm_enabled: false
        llm_model: null
      
      # Future: Plot extraction
      plot_extraction:
        enabled: false
```

---

## 8. Example Public Events to Test

Here are some public CERN Indico events you can test with:

```
# Large conference with many talks
indico-https://indico.cern.ch/category/7389/

# Single workshop event
indico-https://indico.cern.ch/event/1443798/
```

**Note:** Event IDs change - check https://indico.cern.ch/ for current events.

---

## 9. Integration with RAG Pipeline

Once scraped, the markdown content flows into your vector store:

1. **Scraper** → Markdown files in `data/indico/`
2. **Data Manager** → Chunks and embeds markdown
3. **Vector Store** → Stores embeddings with metadata
4. **RAG Query** → Retrieves relevant slides/talks

The hierarchical metadata (event → contribution → material) helps with context-aware retrieval.

---

## 10. Next Steps

- ✅ Test with public events first
- ✅ Add your own event URLs to weblists
- ✅ Enable SSO if needed for protected events
- ✅ Monitor first run to verify conversion quality
- 📊 Future: Add plot extraction hooks for figures in slides

---

## Need Help?

Check the logs for detailed error messages:
```bash
# Logs are typically in the deployment directory
tail -f ~/.a2rchi/a2rchi-<name>/logs/data_manager.log
```

Or run with verbose logging:
```python
import logging
logging.basicConfig(level=logging.INFO)
```

