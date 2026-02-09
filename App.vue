<template>
  <div class="container">
    <header>
      <h1>PDB Download Link Generator</h1>
      <p class="subtitle">Directly generate download links for standard PDB files</p>
    </header>

    <div class="input-section">
      <h2 style="font-size: 14px; color: #5f6368; margin-bottom: 12px; font-weight: 500;">ENTER PDB ID</h2>
      <div class="input-group">
        <label for="proteinPdbIdInput" class="sr-only">Input PDB ID</label>
        <input
            ref="proteinPdbIdInputRef"
            type="text"
            id="proteinPdbIdInput"
            placeholder="e.g. 1TIM, 1A2C, 6LU7"
            autocomplete="off"
            autocapitalize="characters"
            v-model="pdbIdInput"
            @keypress.enter="handleGenerateClick"
        >
        <button id="generateBtn" aria-label="Generate Links" @click="handleGenerateClick">
          Search
        </button>
      </div>

      <div class="examples">
        <span style="font-size: 12px; color: #70757a; align-self: center;">Try:</span>
        <div class="example-chip" @click="handleExampleClick('1TIM')">1TIM</div>
        <div class="example-chip" @click="handleExampleClick('6LU7')">6LU7</div>
        <div class="example-chip" @click="handleExampleClick('4HHB')">4HHB</div>
      </div>
    </div>

    <div class="results-section" :class="{ active: showResults }" ref="resultsSectionRef">
      <div class="result-header">
        <div class="pdb-id-display">PDB ID: {{ generatedPdbId }}</div>
        <p style="color: #5f6368; font-size: 14px; margin-top: 4px;">Links generated below.</p>
      </div>

      <div class="links-grid">
        <div v-for="(link, index) in generatedLinks" :key="index" class="link-card">
          <div class="card-title">{{ link.title }}</div>
          <div class="link-url">{{ link.url }}</div>
          <div class="card-actions">
            <a :href="link.url" target="_blank" :class="link.buttonClass">{{ link.buttonText }}</a>
            <button class="copy-btn" @click="copyToClipboard(link.url, $event)">Copy</button>
          </div>
        </div>
      </div>
    </div>

    <div class="info-section">
      <div class="info-title">About Formats</div>
      <div class="info-content">
        <ul class="info-list" style="list-style-type: none; padding: 0;">
          <li><strong>PDB File</strong>: Standard protein structure file from RCSB.</li>
          <li><strong>.gz File</strong>: Compressed format to save bandwidth.</li>
          <li><strong>CIF/XML</strong>: Alternative data formats.</li>
        </ul>
        <p style="margin-top: 10px; font-size: 12px;"><strong>Note</strong>: Link generation is offline; downloading requires internet.</p>
      </div>
    </div>

    <section class="legal-notice">
      <h3 style="color: #202124; margin-bottom: 8px; font-size: 13px; font-weight: 500;">Disclaimer</h3>
      <p>This tool is a pure front-end utility to generate standardized URLs. It does not host or modify any data. All files are served directly from RCSB PDB.</p>
    </section>
  </div>

  <footer>
    <div style="margin-bottom: 12px;">PDB Link Generator &bull; Offline Secure &bull; {{ currentYear }}</div>
    <div class="author-info" style="font-family: 'Noto Serif SC', 'Source Han Serif SC', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;">
      Developed by Liao H. | Contact: <span v-html="contactEmail"></span>
    </div>
  </footer>
</template>

<script setup>
import { ref, onMounted } from 'vue';

// --- Reactive State ---
const pdbIdInput = ref('');
const generatedPdbId = ref('');
const generatedLinks = ref([]);
const showResults = ref(false);
const currentYear = ref(new Date().getFullYear());
const contactEmail = ref('');

// --- Template Refs ---
const proteinPdbIdInputRef = ref(null);
const resultsSectionRef = ref(null);

// --- Lifecycle Hooks ---
onMounted(() => {
  // Generate contact email
  const parts = ['liaohaocong', 'stu2025','jnu', 'edu', 'cn'];
  const email = parts[0] + '@' + parts[1] + '.' + parts[2] + '.' + parts[3] + '.' + parts[4];
  contactEmail.value = `<a href="mailto:${email}">${email}</a>`;

  // Auto-focus the input field
  proteinPdbIdInputRef.value?.focus();

  // Check for PDB ID in URL parameters
  const urlParams = new URLSearchParams(window.location.search);
  const urlPdbId = (urlParams.get('pdbid') || urlParams.get('id') || '').trim().toUpperCase();
  if (urlPdbId && validatePdbId(urlPdbId)) {
    pdbIdInput.value = urlPdbId;
    setTimeout(() => generateAndShowLinks(urlPdbId), 200); // Small delay for better UX
  }
});


// --- Methods ---

const validatePdbId = (pdbId) => {
  if (!pdbId) {
    alert('Please enter a PDB ID');
    return false;
  }
  // Original Regex: /^[0-9][A-Za-z0-9]{3}$/
  // Loosened the regex to match the original's code intent (which allowed longer IDs, e.g., for AlphaFold)
  if (!/^[0-9][A-Za-z0-9]{3,9}$/.test(pdbId)) {
      alert('Invalid PDB ID format.\nExample: 1TIM, 1A2C');
      return false;
  }
  return true;
};

const generateAllLinks = (pdbId) => {
  return [
    { title: 'Download PDB', url: `https://files.rcsb.org/download/${pdbId}.pdb`, buttonText: 'Download', buttonClass: 'download-btn' },
    { title: 'PDB (.gz)', url: `https://files.rcsb.org/download/${pdbId}.pdb.gz`, buttonText: 'Download .gz', buttonClass: 'download-btn' },
    { title: 'mmCIF Format', url: `https://files.rcsb.org/download/${pdbId}.cif`, buttonText: 'Download CIF', buttonClass: 'download-btn' },
    { title: 'XML Format', url: `https://files.rcsb.org/download/${pdbId}.xml`, buttonText: 'Download XML', buttonClass: 'download-btn' },
    { title: 'RCSB Page', url: `https://www.rcsb.org/structure/${pdbId}`, buttonText: 'Visit RCSB', buttonClass: 'download-btn info-btn' },
    { title: 'PDBe Page', url: `https://www.ebi.ac.uk/pdbe/entry/pdb/${pdbId}`, buttonText: 'Visit PDBe', buttonClass: 'download-btn info-btn' },
    { title: 'UniProt Search', url: `https://www.uniprot.org/uniprotkb?query=${pdbId}`, buttonText: 'Search UniProt', buttonClass: 'download-btn search-btn' },
    { title: 'PDBsum', url: `https://www.ebi.ac.uk/pdbsum/${pdbId}`, buttonText: 'Visit PDBsum', buttonClass: 'download-btn info-btn' },
    { title: 'FASTA', url: `https://www.rcsb.org/fasta/entry/${pdbId}/display`, buttonText: 'View FASTA', buttonClass: 'download-btn' },
    { title: 'Structure Factors', url: `https://files.rcsb.org/download/${pdbId}-sf.cif`, buttonText: 'Download SF', buttonClass: 'download-btn' }
  ];
};

const generateAndShowLinks = (pdbId) => {
  generatedPdbId.value = pdbId;
  generatedLinks.value = generateAllLinks(pdbId);
  showResults.value = true;

  // Scroll to results after they are rendered
  // Using nextTick is a good practice, but for this animation, direct scroll is fine.
  resultsSectionRef.value?.scrollIntoView({ behavior: 'smooth', block: 'start' });
};

const handleGenerateClick = () => {
  const pdbId = pdbIdInput.value.trim().toUpperCase();
  if (validatePdbId(pdbId)) {
    generateAndShowLinks(pdbId);
  }
};

const handleExampleClick = (pdbId) => {
  pdbIdInput.value = pdbId;
  generateAndShowLinks(pdbId);
};

const copyToClipboard = async (text, event) => {
  try {
    await navigator.clipboard.writeText(text);
    const btn = event.target;
    const originalText = btn.textContent;
    btn.textContent = 'Copied';
    btn.classList.add('copy-success');
    setTimeout(() => {
      btn.textContent = originalText;
      btn.classList.remove('copy-success');
    }, 2000);
  } catch (err) {
    console.error('Copy failed: ', err);
    alert('Copy failed');
  }
};

</script>

<style>
/* Google Fonts */
@import url('https://fonts.googleapis.com/css2?family=Noto+Serif+SC:wght@200;300;400&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Roboto+Mono:wght@400;500&display=swap');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

/* Using #app to ensure styles apply correctly within Vue's root element */
#app, body, html {
  font-family: 'Noto Serif SC', 'Source Han Serif SC', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  line-height: 1.5;
  color: #202124;
  background-color: #ffffff;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center; /* Center the container */
}


/* Container */
.container {
  max-width: 800px;
  width: 100%;
  margin: 0 auto;
  padding: 20px;
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
}

/* Header */
header {
  text-align: center;
  padding: 60px 20px 40px;
  width: 100%;
}

h1 {
  font-size: 2.25rem;
  font-weight: 400;
  color: #202124;
  margin-bottom: 8px;
  letter-spacing: -0.5px;
  font-family: 'Noto Serif SC', 'Source Han Serif SC', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
}

.subtitle {
  font-size: 1.1rem;
  color: #5f6368;
  margin-bottom: 10px;
  font-family: 'Noto Serif SC', 'Source Han Serif SC', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
}

/* Google Style Search Bar */
.input-section {
  width: 100%;
  max-width: 584px;
  margin-bottom: 40px;
}

.input-group {
  position: relative;
  width: 100%;
  display: flex;
  align-items: center;
}

input {
  width: 100%;
  height: 48px;
  padding: 10px 110px 10px 20px; /* Space for button */
  border: 1px solid #dfe1e5;
  border-radius: 24px;
  font-size: 16px;
  color: #202124;
  outline: none;
  transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
  font-family: 'Noto Serif SC', 'Source Han Serif SC', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
}

input:hover {
  box-shadow: 0 1px 6px rgba(32,33,36,0.28);
  border-color: rgba(223,225,229,0);
}

input:focus {
  box-shadow: 0 1px 6px rgba(32,33,36,0.28);
  border-color: rgba(223,225,229,0);
}

/* Primary Button inside Input */
button#generateBtn {
  position: absolute;
  right: 4px;
  top: 4px;
  bottom: 4px;
  background-color: #1a73e8;
  color: white;
  border: none;
  padding: 0 24px;
  border-radius: 20px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: background-color 0.2s;
  font-family: 'Roboto Mono', monospace;
}

button#generateBtn:hover {
  background-color: #1557b0;
  box-shadow: 0 1px 2px rgba(60,64,67,0.3);
}

/* Example Chips */
.examples {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-top: 20px;
}

.example-chip {
  background: #f1f3f4;
  color: #3c4043;
  padding: 8px 16px;
  border-radius: 18px;
  font-size: 14px;
  cursor: pointer;
  border: 1px solid transparent;
  transition: background-color 0.2s, border-color 0.2s;
  font-family: 'Noto Serif SC', 'Source Han Serif SC', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
}

.example-chip:hover {
  background: #e8eaed;
  border-color: #dadce0;
}

/* Results Section */
.results-section {
  width: 100%;
  display: none;
  margin-top: 20px;
}

.results-section.active {
  display: block;
  animation: fadeIn 0.4s ease;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.result-header {
  border-bottom: 1px solid #dfe1e5;
  padding-bottom: 16px;
  margin-bottom: 24px;
}

.pdb-id-display {
  font-size: 1.5rem;
  color: #202124;
  font-weight: 400;
  font-family: 'Noto Serif SC', 'Source Han Serif SC', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
}

/* Material Cards Grid */
.links-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 16px;
}

.link-card {
  background: #fff;
  border: 1px solid #dadce0;
  border-radius: 8px;
  padding: 16px;
  transition: box-shadow 0.2s, border-color 0.2s;
  display: flex;
  flex-direction: column;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Arial, sans-serif;
}

.link-card:hover {
  border-color: #dfe1e5;
  box-shadow: 0 4px 12px rgba(32,33,36,0.1);
}

.card-title {
  font-size: 1rem;
  font-weight: 500;
  color: #202124;
  margin-bottom: 8px;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Arial, sans-serif;
}

.link-url {
  font-family: 'Roboto Mono', monospace;
  background: #f8f9fa;
  padding: 8px 12px;
  border-radius: 4px;
  font-size: 12px;
  color: #5f6368;
  margin-bottom: 16px;
  word-break: break-all;
  border: 1px solid #f1f3f4;
}

/* Buttons in Cards */
.card-actions {
  display: flex;
  gap: 8px;
  margin-top: auto;
}

.download-btn {
  flex: 1;
  text-align: center;
  background-color: white;
  color: #1a73e8;
  border: 1px solid #dadce0;
  padding: 8px 16px;
  border-radius: 4px;
  font-size: 14px;
  font-weight: 500;
  text-decoration: none;
  transition: background-color 0.2s, border-color 0.2s;
  font-family: 'Roboto Mono', monospace;
}

.download-btn:hover {
  background-color: #f6fafe;
  border-color: #1a73e8;
}

.download-btn.info-btn, .download-btn.search-btn {
  color: #5f6368;
}

.download-btn.info-btn:hover, .download-btn.search-btn:hover {
  color: #202124;
  background-color: #f1f3f4;
  border-color: #5f6368;
}

.copy-btn {
  background: transparent;
  color: #5f6368;
  border: 1px solid transparent;
  padding: 8px 12px;
  border-radius: 4px;
  font-size: 14px;
  cursor: pointer;
  transition: background 0.2s;
  font-family: 'Roboto Mono', monospace;
}

.copy-btn:hover {
  background: #f1f3f4;
  color: #202124;
}

.copy-success {
  color: #188038 !important; /* Google Green */
  font-weight: 500;
}

/* Info Section */
.info-section {
  background: #f8f9fa;
  border: 1px solid #dfe1e5;
  border-radius: 8px;
  padding: 20px;
  margin-top: 40px;
  width: 100%;
}

.info-title {
  font-size: 16px;
  font-weight: 500;
  color: #202124;
  margin-bottom: 12px;
  font-family: 'Noto Serif SC', 'Source Han Serif SC', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
}

.info-content {
  font-size: 14px;
  color: #5f6368;
  line-height: 1.6;
  font-family: 'Noto Serif SC', 'Source Han Serif SC', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
}

.info-list li {
  margin-bottom: 6px;
}

/* Legal Notice */
.legal-notice {
  font-size: 12px;
  color: #70757a;
  margin-top: 24px;
  padding: 16px;
  border-top: 1px solid #dfe1e5;
  width: 100%;
  font-family: 'Noto Serif SC', 'Source Han Serif SC', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
}

/* Footer */
footer {
  background: #FFFFFF;
  padding: 24px;
  text-align: center;
  border-top: 1px solid #e4e4e4;
  width: 100%;
  font-size: 13px;
  color: #70757a;
  font-family: 'Noto Serif SC', 'Source Han Serif SC', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
}

footer a {
  color: #70757a;
  text-decoration: none;
}

footer a:hover {
  text-decoration: underline;
}

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

@media (max-width: 600px) {
  header { padding: 40px 20px 20px; }
  h1 { font-size: 1.75rem; }
  input { font-size: 16px; }
}
</style>
