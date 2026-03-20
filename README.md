# Huayang Isolator PDF Extractor & Generator
A Node.js-based application designed to extract product data from Huayang Communication's isolator/circulator product manuals (PDF format) and generate individual, standardized PDF datasheets for each product variant. The application addresses the unique challenge of splitting multi-product summary tables into distinct, properly-named PDF documents while preserving shared visual assets (graphs, diagrams) and ensuring data accuracy.

## Key Features
- **Multi-Product Splitting**: Automatically identifies and splits aggregated product tables (multiple variants per page) into individual PDF datasheets
- **Intelligent Naming Convention**: Enforces consistent product ID formatting with custom rules for models with/without explicit "Model" columns
- **Visual Asset Management**: Extracts and rearranges graphs/diagrams in a clean 2x2 grid layout (scaled appropriately, no boundary overflow)
- **Data Sanitization**: Eliminates garbled characters (non-ASCII noise) from technical tables and ensures consistent text formatting
- **Branded Output**: Generates PDFs with standardized branding (RFecho trademark, copyright, website URL) and professional dark-gray theme
- **Bulk Processing**: Supports batch upload of manual folders for large-scale product extraction

## Prerequisites
- Node.js (v16+ recommended)
- Gemini API Key (for PDF text/image extraction)

## Installation & Usage

### 1. Clone the Repository
```bash
git clone https://github.com/your-username/huayang-isolator-pdf-generator.git
cd huayang-isolator-pdf-generator
```

### 2. Install Dependencies
```bash
npm install
```

### 3. Configure Environment Variables
Create a `.env.local` file in the root directory and add your Gemini API key:
```env
GEMINI_API_KEY=your_gemini_api_key_here
```

### 4. Run the Application
```bash
npm run dev
```

### 5. Use the Application
1. Access the local development server (typically `http://localhost:3000`)
2. Upload Huayang Communication's product manual PDF(s) (e.g., "深圳市华扬通信技术有限公司产品手册·2025版.pdf")
3. The application will:
   - Extract product data from summary tables
   - Split multi-row frequency variants into individual products
   - Generate sanitized technical tables (no garbled characters)
   - Rearrange graphs/diagrams in a 2x2 grid layout
   - Create branded PDF datasheets for each product variant
4. Download the generated PDFs (automatically named per the custom convention)

## Product Naming Rules
The application enforces strict naming conventions for generated PDFs and internal product IDs:

### Case 1: Table has "Model" column
- Format: `O<Model>-<Frequency Range>` (last hyphen in frequency range remains a hyphen)
- Example: `H103E-1805-1880` → `OH103E-1805-1880`

### Case 2: Table has NO "Model" column
- Remove special characters (e.g., Φ, ф) from size specifications
- Format: `O-<Numeric Size>-<Frequency Range>`
- Example: `Φ5mm-3400-3600` → `O-5mm-3400-3600`

## Customization Guide
Below are detailed instructions for modifying core functionality, styling, and business logic:

### 1. Modify Product Naming Rules
**File to Edit**: `index.tsx`  
**Location**: Search for the `formatProductId` (or similar) function (typically a helper function for ID generation)

#### Example Changes:
```typescript
// Existing function (simplified)
const formatProductId = (model: string | null, size: string, frequency: string) => {
  let baseId = "";
  if (model) {
    // Case 1: Has model column
    baseId = `O${model}-${frequency}`;
  } else {
    // Case 2: No model column - remove special chars
    const cleanSize = size.replace(/[Φф]/g, "");
    baseId = `O-${cleanSize}-${frequency}`;
  }
  // Keep hyphens (no underscore replacement)
  return baseId;
};

// To change prefix from "O" to "HY":
const formatProductId = (model: string | null, size: string, frequency: string) => {
  let baseId = "";
  if (model) {
    baseId = `HY${model}-${frequency}`;
  } else {
    const cleanSize = size.replace(/[Φф]/g, "");
    baseId = `HY-${cleanSize}-${frequency}`;
  }
  return baseId;
};
```

### 2. Adjust PDF Styling (Colors/Layout)
#### A. Change Theme Colors (Dark/Gray → Custom)
**File to Edit**: `index.tsx`  
**Location**: PDF generation section (look for CSS/styles in the PDF creator logic)
```typescript
// Existing dark-gray theme styles (simplified)
const pdfStyles = {
  header: {
    color: "#333333", // Dark gray
    backgroundColor: "#F5F5F5", // Light gray
  },
  table: {
    borderColor: "#666666", // Medium gray
    headerColor: "#444444", // Dark gray
  }
};

// Modify to blue theme example:
const pdfStyles = {
  header: {
    color: "#FFFFFF", 
    backgroundColor: "#0056b3", // Primary blue
  },
  table: {
    borderColor: "#007bff", // Light blue
    headerColor: "#004085", // Dark blue
  }
};
```

#### B. Adjust Graph/Image Layout
**File to Edit**: `index.tsx`  
**Location**: Image processing/rendering function (search for `renderGraphs` or `layoutImages`)
```typescript
// Existing 2x2 grid (75mm height)
const renderGraphs = (images: string[]) => {
  const graphSize = {
    height: 75, // mm
    width: 140, // mm (fits 2 per row on A4)
  };
  const rows = chunk(images, 2); // Split into 2 per row
  
  // Render rows
  rows.forEach(row => {
    // Add row with 2 images
  });
};

// Change to 1x3 grid (smaller images)
const renderGraphs = (images: string[]) => {
  const graphSize = {
    height: 50, // mm (smaller)
    width: 100, // mm (fits 3 per row)
  };
  const rows = chunk(images, 3); // Split into 3 per row
  
  // Render rows
  rows.forEach(row => {
    // Add row with 3 images
  });
};
```

### 3. Update PDF Branding (Footer/Header)
**File to Edit**: `index.tsx`  
**Location**: PDF header/footer generation section
```typescript
// Existing footer config
const pdfFooter = {
  left: "www.rfecho.com",
  right: [
    "RFecho is trademark of Ocean Microwave",
    "©RFecho 2025"
  ]
};

// Modify to new branding example:
const pdfFooter = {
  left: "www.huayang-communication.com",
  right: [
    "Huayang Communication ©2025",
    "All Rights Reserved"
  ]
};

// Change "ID" label back to "Model" (top-right)
const pdfHeader = {
  right: "ID: {productId}" // Change to "Model: {productId}"
};
```

### 4. Fix Garbled Characters (Add More Symbols)
**File to Edit**: `index.tsx`  
**Location**: `sanitizeText` function (text cleaning logic)
```typescript
// Existing sanitization
const sanitizeText = (text: string) => {
  // Preserve essential symbols, remove garbled chars
  return text
    .replace(/[Æø‰Ä¦]/g, "") // Remove known garbage
    .replace(/[Φф]/g, "") // Remove size special chars
    // Preserve technical symbols
    .replace(/°|µ|Ω|±|≤|≥/g, match => match);
};

// Add more garbage characters to filter
const sanitizeText = (text: string) => {
  return text
    .replace(/[Æø‰Ä¦ßãð]/g, "") // Add new garbage chars here
    .replace(/[Φф]/g, "")
    .replace(/°|µ|Ω|±|≤|≥/g, match => match);
};
```

### 5. Modify Description Text Format
**File to Edit**: `index.tsx`  
**Location**: AI prompt section (Gemini instruction string) and description generation
```typescript
// Existing AI prompt for description
const geminiPrompt = `
  Generate a product description starting with:
  "The product named "{productId}" is high-performance..."
`;

// Change description format
const geminiPrompt = `
  Generate a product description starting with:
  "Isolator {productId} is a premium-grade component designed for..."
`;
```

### 6. Adjust PDF Table Headers (Technical Indicators)
**File to Edit**: `index.tsx`  
**Location**: Table header mapping function (search for "Technical indicators" or "Size Spec")
```typescript
// Existing header mapping
const headerMappings = {
  "尺寸规格": "Size Spec",
  "工作频率": "Freq. (MHz)",
  "回波损耗": "Return Loss"
};

// Add/modify headers
const headerMappings = {
  "尺寸规格": "Dimensions", // Rename Size Spec → Dimensions
  "工作频率": "Operating Frequency (MHz)", // More descriptive
  "插入损耗": "Insertion Loss (dB)", // Add new mapping
  "回波损耗": "Return Loss"
};
```

## Troubleshooting
### Common Issues
1. **Document Size Exceeds Limit**: Reduce PDF resolution in the image processing step (lower scaling factor from 3.0 to 2.0)
2. **Missing Products**: Verify the AI prompt explicitly instructs to split every table row into a separate product
3. **Graph Overflow**: Adjust `graphSize` dimensions in the image layout function (reduce width/height)
4. **API Key Errors**: Ensure `.env.local` is correctly formatted and Gemini API key has sufficient quota

### Debug Tips
- Check the browser console for frontend errors
- Verify the Gemini API response (log raw data) to ensure correct product extraction
- Test with smaller PDF files first (single page) to isolate issues

## License
This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments
- Built with Node.js and Gemini API for PDF text/image extraction
- Optimized for Huayang Communication's isolator/circulator product manuals
- Designed for industrial-grade PDF datasheet generation
