# Indico Test Deployment

Test deployment for the new Indico event scraping feature.

## Quick Start

### 1. Configure Secrets

```bash
# Edit secrets.env and add your OpenAI API key
nano test-indico-deployment/secrets.env
```

At minimum, set:
```
OPENAI_API_KEY=sk-...
```

For protected events, also set:
```
SSO_USERNAME=your-cern-username
SSO_PASSWORD=your-cern-password
```

### 2. Add Indico Event URLs

Edit `indico_events.list` and add events you want to scrape:

```bash
nano test-indico-deployment/indico_events.list
```

Format:
```
indico-https://indico.cern.ch/event/123456/
```

### 3. Deploy

```bash
archi create indico-test \
  --config-dir test-indico-deployment \
  --env-file test-indico-deployment/secrets.env \
  --sources indico \
  --services chatbot
```

### 4. Monitor

```bash
# Check status
archi status -n indico-test

# Watch data collection logs
tail -f ~/.archi/archi-indico-test/logs/data_manager.log

# Check chat interface
open http://localhost:5000
```

## What Gets Scraped

For each Indico URL:
- ✅ **Event metadata**: Title, description, dates, location
- ✅ **Contribution metadata**: Speaker names, talk titles, abstracts
- ✅ **Slides**: PDF, PPTX, PPT, ODP converted to markdown
- ✅ **Hierarchical metadata**: Links between events, talks, and materials

## Configuration

Main config options in `config.yaml`:

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

## Testing Without Full Deployment

If you just want to test the scraper without the full stack:

```bash
# Activate environment
source archi-env/bin/activate

# Run standalone test
python test_indico_working.py
```

## Troubleshooting

### "No materials found"
- Not all events have uploaded slides
- Check the event page manually to verify materials exist
- Try a different event

### "Authentication failed"
- Check SSO_USERNAME and SSO_PASSWORD are correct
- Try with `use_sso: false` and a public event first

### "ModuleNotFoundError"
- Make sure you're in the virtual environment: `source archi-env/bin/activate`
- Install dependencies: `pip install -r requirements/requirements-base.txt`

### Port already in use
- Change `external_port` in config.yaml
- Or stop existing deployment: `archi stop -n indico-test`

## Next Steps

Once data is collected:
1. Open chat interface: http://localhost:5000
2. Ask questions about the scraped events/slides
3. Check retrieved sources show "indico" as source_type
4. Verify markdown conversion quality

## Clean Up

```bash
# Stop deployment
archi stop -n indico-test

# Remove deployment
archi remove -n indico-test
```

