## 📋 Tasks
<%*
const MAX_DAYS_BACK = 14; // safety limit
let foundFile = null;

for (let i = 1; i <= MAX_DAYS_BACK; i++) {
  const date = tp.date.now("YYYY-MM-DD", -i);
  const path = `Daily/${date}.md`;
  const file = app.vault.getAbstractFileByPath(path);

  if (file) {
    foundFile = file;
    break;
  }
}

if (foundFile) {
  const content = await app.vault.read(foundFile);
  const match = content.match(
    /### What to do tomorrow:\n([\s\S]*?)(\n##|\s*$)/
  );

  if (match) {
    tR += match[1].trim();
  }
}
%>

## 📌 Notes
- 

## 🌙 End of Day Review

### What to do tomorrow:
- [ ] 
