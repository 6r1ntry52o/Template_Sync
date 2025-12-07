
```dataviewjs
// ターゲット（＝このファイル）
const current = dv.current().file; 
const targetName = current.name;
const targetPathNoExt = current.path.replace(/\.md$/i, "");
const currentFolder = current.folder;

// このディレクトリ内のファイル一覧
const filesUnderFolder = dv.pages()
  .where(p => p.file && p.file.path && p.file.folder.startsWith(currentFolder))
  .map(p => {
    return {
      name: p.file.name,
      nameNoExt: p.file.name.replace(/\.md$/i, ""),
      path: p.file.path,
      pathNoExt: p.file.path.replace(/\.md$/i, "")
    };
  });

// そのファイルへのリンクを持つノート一覧（ここは元のまま）
const pages = dv.pages()
  .where(p =>
    p.file.folder !== currentFolder &&
    p.file.outlinks &&
    p.file.outlinks.some(out =>
      filesUnderFolder.some(f =>
        out.path === f.path || out.path === f.path.replace(/\.md$/i, "")
      )
    )
  );

// 行を分類する関数
function classifyLine(line) {
  if (/^\s*-\s*\[\s*\]\s+/.test(line)) return "⭕";
  if (/^\s*-\s*\[[xX]\]\s+/.test(line)) return "✅";
  if (/^\s*-\s+/.test(line)) return "-";
  return "その他";
}

let results = [];

for (let page of pages) {
  const raw = await dv.io.load(page.file.path);
  const lines = raw.split("\n");
  
  let block = [];
  
  const pushBlock = () => {
    if (block.length === 0) return;
    
    let time = ""; 
    let due = "";
    let done = "";
    let bodyLines = [...block];
    const firstLine = bodyLines[0] ?? "";
    
    const timeMatch = firstLine.match(/\b\d{1,2}[:：]\d{2}\b/);
    if (timeMatch) time = timeMatch[0];
    
    const dateMatch = firstLine.match(/📅\s*(\d{4}-\d{2}-\d{2})/);
    if (dateMatch) due = dateMatch[1];
    
    const doneMatch = firstLine.match(/✅\s*(\d{4}-\d{2}-\d{2})/);
    if (doneMatch) done = doneMatch[1];
    
    const prefixMatch = firstLine.match(/^(\s*-\s*(?:\[[ xX]\]\s*)?)/);
    const prefix = prefixMatch ? prefixMatch[1] : "";
    const type = classifyLine(prefix);
    
    let content = prefixMatch ? firstLine.slice(prefix.length) : firstLine;
    
    content = content
      .replace(/\b\d{1,2}[:：]\d{2}\b/, "")
      .replace(/📅\s*\d{4}-\d{2}-\d{2}/, "")
      .replace(/✅\s*\d{4}-\d{2}-\d{2}/, "")
      .replace(/\s{2,}/g, " ")
      .trim();
      
    bodyLines[0] = `${content}`;
    
    const filter = bodyLines.join("\n");
    const filterText = String(filter ?? "");
    
    const text = bodyLines.join("<br>")
      .replace(
        new RegExp(`\\[\\[(?:${targetName}|${targetPathNoExt})(?:\\|([^\\]]+))?\\]\\]`, "g"),
        (_, label) => label ?? ""
      )
      .replace(/[ \t]+$/gm, "")
      .trim();
      
    const dateString = page.file.name.replace(/\.md$/i, "");
    const timeString = time || "00:00";
    
    const [year, month, day] = dateString.split("-").map(Number);
    let hh = 0, mm = 0;
    
    if (timeString.match(/^\d{1,2}[:：]\d{2}$/)) {
      [hh, mm] = timeString.replace("：", ":").split(":").map(Number);
    }
    
    const dateObj = new Date(year, month - 1, day, hh, mm);
    
    // ★ ここを無条件で push するように変更 ★
    results.push([
      page.file.link,
      time,
      dv.el("div", text, { cls: "keep-indent" }),
      dateObj,
      due,
      done,
      type
    ]);

    block = [];
  };
  
  for (let line of lines) {
    if (line.match(/^-\s*(\[[ x]\])?/) || line.match(/^[\*\+]\s+/)) {
      if (!line.match(/^\s+/)) {
        pushBlock();
        block.push(line);
        continue;
      }
    }
    if (block.length > 0 && line.match(/^\s+/)) {
      block.push(line);
      continue;
    }
  }
  pushBlock();
}

results.sort((a, b) => b[3] - a[3]);

dv.table(
  ["Date", "Time", "", "What I was thinking", "📅", "✅"],
  results.map(r => [
    r[0],
    r[1],
    r[6],
    r[2],
    r[4] ? r[4] : "`-`",
    r[5] ? r[5] : "`-`"
  ])
);

```
