CSVファイルのレイアウト
ID,IF名,ソースエンティティ,ターゲットエンティティ,主管部署,個人情報の有無
1,IF001,エンティティA,エンティティB,部署X,なし
2,IF002,エンティティB,エンティティC,部署Y,あり
3,IF003,エンティティC,エンティティD,部署Z,なし
4,IF004,エンティティD,エンティティE,部署X,なし
5,IF005,エンティティF,エンティティG,部署Y,あり

mermaid記法でのシステム間構成図の例
graph TD;
    エンティティA -->|IF001| エンティティB;
    エンティティB -->|IF002| エンティティC;
    エンティティC -->|IF003| エンティティD;
    エンティティD -->|IF004| エンティティE;
    エンティティF -->|IF005| エンティティG;

PowerShellでCSVファイルからmermaid記法のマークダウンを作成するプログラム
# CSVファイルのパス
$csvPath = "C:\path\to\csv\file.csv"

# mermaid記法のマークダウンを保存するファイルのパス
$mdPath = "C:\path\to\output\file.md"

# CSVファイルを読み込む
$csv = Import-Csv $csvPath

# mermaid記法のマークダウンを作成する
$mermaid = "graph TD;" + [Environment]::NewLine
foreach ($row in $csv) {
    $sourceEntity = $row.'ソースエンティティ'
    $targetEntity = $row.'ターゲットエンティティ'
    $ifName = $row.'IF名'

    $mermaid += "    $sourceEntity -->|$ifName| $targetEntity;" + [Environment]::NewLine
}

# マークダウンをファイルに保存する
Set-Content -Path $mdPath -Value $mermaid

Next.jsをつかったアプリケーションをつくります。
要件は以下になります。
①CSVをアップロード（CSVのレイアウトはさきほどのシステム関連携と同じです）
②アップロードしたCSVからデータフローダイヤグラムを描画
③画面上でネットワーク図を更新できる
④更新したデータフローダイヤグラムをCSVでダウンロード

import { useState } from 'react';
import ReactFlow from 'react-flow-renderer';
import { saveAs } from 'file-saver';

const App = () => {
  const [elements, setElements] = useState([]);

  const handleFileChange = (event) => {
    const file = event.target.files[0];
    const reader = new FileReader();
    reader.onload = () => {
      const csvData = reader.result;
      const lines = csvData.split('\n');
      const newElements = lines.map((line, index) => {
        const data = line.split(',');
        return {
          id: index.toString(),
          data: { label: data[1] },
          position: { x: parseInt(data[2]), y: parseInt(data[3]) },
        };
      });
      setElements(newElements);
    };
    reader.readAsText(file);
  };

  const handleNodeDrag = (event, node) => {
    node.position = { x: event.clientX, y: event.clientY };
    setElements([...elements]);
  };

  const handleDownload = () => {
    const csvData = elements
      .map((element) => `${element.id},${element.data.label},${element.position.x},${element.position.y}`)
      .join('\n');
    const blob = new Blob([csvData], { type: 'text/csv;charset=utf-8;' });
    saveAs(blob, 'data.csv');
  };

  return (
    <div>
      <form>
        <input type="file" onChange={handleFileChange} />
      </form>
      <ReactFlow elements={elements} onNodeDragStop={handleNodeDrag} />
      <button onClick={handleDownload}>Download CSV</button>
    </div>
  );
};

export default App;




