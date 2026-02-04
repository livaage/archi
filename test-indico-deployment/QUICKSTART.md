# Quick Start Guide: Indico Scraper Test

## 📁 What Was Created

```
test-indico-deployment/
├── config.yaml              # Complete deployment configuration
├── secrets.env              # API keys and credentials (EDIT THIS!)
├── indico_events.list       # List of Indico URLs to scrape
├── prompts/                 # Prompt templates
│   ├── condense/default.prompt
│   ├── chat/default.prompt
│   └── system/default.prompt
├── README.md                # Detailed documentation
└── QUICKSTART.md           # This file
```

## 🚀 Deploy in 3 Steps

### Step 1: Add Your API Key

```bash
# Edit secrets.env
nano test-indico-deployment/secrets.env

# Add at minimum:
OPENAI_API_KEY=sk-your-key-here
```

### Step 2: (Optional) Add More Events

```bash
# Edit the event list
nano test-indico-deployment/indico_events.list

# Format: indico-<full-url>
indico-https://indico.cern.ch/event/123456/
```

### Step 3: Deploy!

```bash
archi create indico-test \
  --config-dir test-indico-deployment \
  --env-file test-indico-deployment/secrets.env \
  --sources indico \
  --services chatbot
```

## ✅ What to Expect

1. **Deployment creates**: `~/.archi/archi-indico-test/`
2. **Data manager starts**: Scrapes Indico events
3. **Slides converted**: PDF/PPTX → Markdown
4. **Vector store populated**: Events and slides indexed
5. **Chat interface ready**: http://localhost:5000

## 📊 Monitor Progress

```bash
# Check overall status
archi status -n indico-test

# Watch scraping in real-time
tail -f ~/.archi/archi-indico-test/logs/data_manager.log

# See what was collected
ls ~/.archi/archi-indico-test/data/indico/
```

## 💬 Test the Chat

Once deployed, open http://localhost:5000 and try:
- "What events have been indexed?"
- "Summarize the talks about [topic]"
- "Who presented on [subject]?"
- "Show me slides about [specific topic]"

## 🔍 Verify It's Working

```bash
# Check if events were scraped
ls ~/.archi/archi-indico-test/data/indico/*.md

# Should see files like:
# - event_123456.md (event metadata)
# - contribution_789.md (talk metadata)  
# - slides_xyz.md (converted slides)
```

## 🛑 Stop/Remove

```bash
# Stop services
archi stop -n indico-test

# Remove completely
archi remove -n indico-test
```

## 🐛 Troubleshooting

### "Command not found: archi"
```bash
# Make sure you installed the package
pip install -e .

# Or activate the environment
source archi-env/bin/activate
```

### "Port 5000 already in use"
Edit `config.yaml` and change:
```yaml
services:
  chatbot:
    external_port: 5001  # or any free port
```

### "No materials found"
- This is normal - not all events have slides
- Try a different event from https://indico.cern.ch/
- Look for events with "Materials" tab

### Need help finding events with materials?
```bash
# Run the test script
python test_indico_working.py
```

## 📚 More Info

- Full documentation: `test-indico-deployment/README.md`
- Setup guide: `INDICO_SETUP_GUIDE.md`
- Spec document: `openspec/specs/indico-scraper.md`

---

**Ready?** Run the Step 1-3 commands above! 🎉

