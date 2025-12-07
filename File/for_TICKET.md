---
created: <% tp.date.now("YYYY-MM-DD") %>
tags:
  - Ticket
Completed: false
Sub: false
---

```dataviewjs
const current = dv.current().file.folder;
const pages = dv.pages().where(p => p.file.folder.startsWith(current));

dv.table(
  ["File","Update"],
  pages.map(p => [
    // 表示名をフォルダ基準の相対パスに変換
    dv.fileLink(
      p.file.path,
      false,
      p.file.path.replace(current + "/", "")
    ),
    p.file.mtime
  ])
);
```
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

// そのファイルへのリンクを持つノート一覧
const pages = dv.pages()
  .where(p =>
    p.file.folder !== currentFolder && // フォルダ外のノートのみ
    p.file.outlinks &&
    p.file.outlinks.some(out =>
      filesUnderFolder.some(f =>
        out.path === f.path || out.path === f.path.replace(/\.md$/i, "")
      )
    )
  );

// 行を分類する関数
function classifyLine(line) {
  if (/^\s*-\s*\[\s*\]\s+/.test(line)) {
    return "⭕";
  }
  if (/^\s*-\s*\[[xX]\]\s+/.test(line)) {
    return "✅";
  }
  if (/^\s*-\s+/.test(line)) {
    return "-";
  }
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
    
	// 先頭行に HH:MM または HH：MM があれば抽出
	const timeMatch = firstLine.match(/\b\d{1,2}[:：]\d{2}\b/);
	if (timeMatch) {
	  time = timeMatch[0];   // "HH:MM"
	}
	
	// 先頭行に 📅 yyyy-mm-dd があれば抽出
	const dateMatch = firstLine.match(/📅\s*(\d{4}-\d{2}-\d{2})/);
	if (dateMatch) {
	  due = dateMatch[1];    // "2025-12-06" の部分だけ取り出す
	}

  // 先頭行に ✅ yyyy-mm-dd があれば抽出
	const doneMatch = firstLine.match(/✅\s*(\d{4}-\d{2}-\d{2})/);
	if (doneMatch) {
	  done = doneMatch[1];    // "2025-12-06" の部分だけ取り出す
	}
	
// 先頭行のプレフィックス（- , - [ ] , - [x] など）を保持
const prefixMatch = firstLine.match(/^(\s*-\s*(?:\[[ xX]\]\s*)?)/);
const prefix = prefixMatch ? prefixMatch[1] : "";
const type = classifyLine(prefix);

// プレフィックス以降の本文だけを抽出
let content = prefixMatch ? firstLine.slice(prefix.length) : firstLine;

// 本文から時刻・📅 yyyy-mm-dd・✅ yyyy-mm-dd を削除
content = content
  .replace(/\b\d{1,2}[:：]\d{2}\b/, "")        // 時刻削除
  .replace(/📅\s*\d{4}-\d{2}-\d{2}/, "")       // 📅 yyyy-mm-dd 削除
  .replace(/✅\s*\d{4}-\d{2}-\d{2}/, "")       // ✅ yyyy-mm-dd 削除
  .replace(/\s{2,}/g, " ")                    // 連続スペース整形
  .trim();

// 最後に元のタスク記法を復元
// bodyLines[0] = `${prefix}${content}`;
bodyLines[0] = `${content}`;

  const filter = bodyLines.join("\n");
    
	// filter を必ず文字列化（undefined 対策）
	const filterText = String(filter ?? "");
	
	// このディレクトリ以下（サブフォルダ含む）ファイルへのリンク判定
	const hasLinkInFilter = filesUnderFolder.some(f =>
	  filterText.includes("[[" + f.name + "]]") ||
	  filterText.includes("[[" + f.nameNoExt + "]]") ||
	  filterText.includes("[[" + f.path + "]]") ||
	  filterText.includes("[[" + f.pathNoExt + "]]")
	);
	
	const text = bodyLines.join("<br>")
	  // 自ファイルへのリンクだけ除去（Markdown構造を壊さない）
	  .replace(
	    new RegExp(`\\[\\[(?:${targetName}|${targetPathNoExt})(?:\\|([^\\]]+))?\\]\\]`, "g"),
	    (_, label) => label ?? ""
	  )
	  // 不要な全角・半角スペースだけ最小限除去（改行は保持）
	  .replace(/[ \t]+$/gm, "")  // 行末の余分な空白のみ削除
	  .trim();
	  
    // このディレクトリのファイルへのリンクがある場合だけ追加
    if (
    hasLinkInFilter
    ) {
	const dateString = page.file.name.replace(/\.md$/i, ""); // ← YYYY-MM-DD のみ
	const timeString = time || "00:00";
	
	// YYYY-MM-DD を分解
	const [year, month, day] = dateString.split("-").map(n => Number(n));
	
	// HH:MM を分解（時刻が無い場合も安全）
	let hh = 0, mm = 0;
	if (timeString.match(/^\d{1,2}[:：]\d{2}$/)) {
	  [hh, mm] = timeString.replace("：", ":").split(":").map(n => Number(n));
	}
	
	// Date オブジェクト生成（安全）
	const dateObj = new Date(year, month - 1, day, hh, mm);
		  
  results.push([
  page.file.link,                        // 日付列
  time,                                  // 
  dv.el("div", text, { cls: "keep-indent" }), // ブロック内容
  dateObj,
  due,
  done,
  type
  ]);
  }

    block = [];
  };

  for (let line of lines) {

    // ■ ブロック開始
    if (
      line.match(/^-\s*(\[[ x]\])?/) ||
      line.match(/^[\*\+]\s+/)
    ) {
      if (!line.match(/^\s+/)) {
        pushBlock();
        block.push(line);
        continue;
      }
    }

    // ■ インデント行 → 継続
    if (block.length > 0 && line.match(/^\s+/)) {
      block.push(line);
      continue;
    }
  }

  pushBlock();
}

// --- ソート処理を追加 ---
results.sort((a, b) => b[3] - a[3]);

dv.table(
  ["Date", "Time", "", "What I was thinking", "📅", "✅"],
  results.map(r => [
    r[0],
    r[1],
    r[6],
    r[2],
    r[4] ? r[4] : "`-`",
    r[5] ? r[4] : "`-`"
  ])
);
 // ,r[3].toISOString()
```
