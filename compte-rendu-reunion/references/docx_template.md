# Template DOCX pour Compte Rendu de RÃ©union

Ce fichier contient le template Node.js (librairie `docx`) pour gÃ©nÃ©rer le compte rendu. Le script doit Ãªtre adaptÃ© Ã  chaque rÃ©union mais la structure reste identique.

## DÃ©pendances

```bash
npm init -y && npm install docx
```

## Structure du script

Le script utilise la librairie `docx` pour Node.js. Voici les imports nÃ©cessaires et les helpers rÃ©utilisables.

### Imports

```javascript
const { Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell,
        Header, Footer, AlignmentType, HeadingLevel, BorderStyle, WidthType,
        ShadingType, PageNumber, LevelFormat, TableOfContents } = require('docx');
const fs = require('fs');
```

### Helpers pour les tableaux

```javascript
const border = { style: BorderStyle.SINGLE, size: 1, color: "999999" };
const borders = { top: border, bottom: border, left: border, right: border };
const cellMargins = { top: 60, bottom: 60, left: 100, right: 100 };

function headerCell(text, width) {
  return new TableCell({
    borders,
    width: { size: width, type: WidthType.DXA },
    shading: { fill: "1F3864", type: ShadingType.CLEAR },
    margins: cellMargins,
    verticalAlign: "center",
    children: [new Paragraph({
      alignment: AlignmentType.LEFT,
      children: [new TextRun({ text, bold: true, font: "Arial", size: 20, color: "FFFFFF" })]
    })]
  });
}

function cell(text, width, opts = {}) {
  return new TableCell({
    borders,
    width: { size: width, type: WidthType.DXA },
    shading: opts.shading ? { fill: opts.shading, type: ShadingType.CLEAR } : undefined,
    margins: cellMargins,
    children: [new Paragraph({
      alignment: AlignmentType.LEFT,
      children: [new TextRun({ text, font: "Arial", size: 20, bold: opts.bold || false })]
    })]
  });
}

function makeRow(cells) {
  return new TableRow({ children: cells });
}
```

### Structure du document

```javascript
const doc = new Document({
  features: { updateFields: true }, // pour le sommaire
  styles: {
    default: { document: { run: { font: "Arial", size: 22 } } },
    paragraphStyles: [
      {
        id: "Heading1", name: "Heading 1", basedOn: "Normal", next: "Normal",
        quickFormat: true,
        run: { size: 28, bold: true, font: "Arial", color: "1F3864" },
        paragraph: { spacing: { before: 300, after: 150 } }
      },
      {
        id: "Heading2", name: "Heading 2", basedOn: "Normal", next: "Normal",
        quickFormat: true,
        run: { size: 24, bold: true, font: "Arial", color: "2E75B6" },
        paragraph: { spacing: { before: 240, after: 120 } }
      },
    ]
  },
  numbering: {
    config: [{
      reference: "bullets",
      levels: [{
        level: 0, format: LevelFormat.BULLET, text: "\u2022",
        alignment: AlignmentType.LEFT,
        style: { paragraph: { indent: { left: 720, hanging: 360 } } }
      }]
    }]
  },
  sections: [{
    properties: {
      page: {
        size: { width: 11906, height: 16838 },
        margin: { top: 1200, right: 1200, bottom: 1200, left: 1200 }
      }
    },
    headers: {
      default: new Header({
        children: [new Paragraph({
          alignment: AlignmentType.RIGHT,
          children: [new TextRun({
            text: "ENTREPRISE  |  RÃ©fÃ©rence Projet",
            font: "Arial", size: 16, color: "888888", italics: true
          })]
        })]
      })
    },
    footers: {
      default: new Footer({
        children: [new Paragraph({
          alignment: AlignmentType.CENTER,
          children: [
            new TextRun({ text: "Page ", font: "Arial", size: 16, color: "888888" }),
            new TextRun({ children: [PageNumber.CURRENT], font: "Arial", size: 16, color: "888888" })
          ]
        })]
      })
    },
    children: [
      // TITRE
      new Paragraph({
        alignment: AlignmentType.CENTER, spacing: { after: 100 },
        children: [new TextRun({
          text: "COMPTE RENDU DE RÃUNION",
          bold: true, font: "Arial", size: 36, color: "1F3864"
        })]
      }),
      new Paragraph({
        alignment: AlignmentType.CENTER, spacing: { after: 300 },
        children: [new TextRun({
          text: "Sous-titre projet",
          font: "Arial", size: 26, color: "2E75B6"
        })]
      }),

      // SOMMAIRE
      new Paragraph({
        heading: HeadingLevel.HEADING_1,
        children: [new TextRun("Sommaire")]
      }),
      new TableOfContents("Sommaire", {
        hyperlink: true,
        headingStyleRange: "1-2"
      }),
      new Paragraph({ spacing: { after: 200 } }),

      // TABLEAU D'INFORMATIONS
      new Table({
        width: { size: 9506, type: WidthType.DXA },
        columnWidths: [2800, 6706],
        rows: [
          makeRow([headerCell("Date", 2800), cell("XX mois YYYY - HHhMM", 6706)]),
          makeRow([headerCell("Type", 2800), cell("RÃ©union Teams / prÃ©sentiel / etc.", 6706)]),
          makeRow([headerCell("Participants", 2800), cell("PrÃ©nom NOM (SociÃ©tÃ©), ...", 6706)]),
          makeRow([headerCell("RÃ©fÃ©rence projet", 2800), cell("REF - Description", 6706)]),
          makeRow([headerCell("RÃ©dacteur", 2800), cell("Entreprise", 6706)]),
        ]
      }),

      // SECTIONS NUMEROTEES (heading: HeadingLevel.HEADING_1)
      // Chaque section est un thÃ¨me abordÃ©
      // Utiliser des puces (numbering bullets) pour les points
      // Utiliser HeadingLevel.HEADING_2 pour les sous-sections

      // TABLEAU D'ACTIONS (toujours en derniÃ¨re section numÃ©rotÃ©e)
      // Colonnes : NÂ° | Action | Responsable | ÃchÃ©ance
    ]
  }]
});

// GÃ©nÃ©ration du fichier
Packer.toBuffer(doc).then(buffer => {
  fs.writeFileSync("OUTPUT_PATH.docx", buffer);
  console.log("DOCX created successfully");
});
```

## RÃ¨gles de formatage

- Pas de tirets longs (â) : utiliser ";" ou " - "
- Toujours PrÃ©nom NOM (jamais juste le nom)
- Texte courant en Arial 11pt (size: 22 en half-points)
- Listes Ã  puces avec bullet "â¢"
- SÃ©paration entre les sections
- Points importants en gras via TextRun({ bold: true })
- Le sommaire nÃ©cessite `features: { updateFields: true }` dans le Document et un `TableOfContents` dans les children. Le sommaire sera mis Ã  jour Ã  l'ouverture du fichier dans Word.
