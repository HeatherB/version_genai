# Seattle EcoPlanner: AI-Assisted Urban Development
## Case Study: Balancing Housing Growth with Ecological Preservation

---

## 🎯 Problem

### The Challenge
Seattle faces a critical urban planning dilemma: **how to address a severe housing crisis without destroying the city's ecological assets**.

**The Housing Reality:**
- Rapid population growth driving demand for affordable homes
- Limited available land for new development
- Pressure to increase housing density

**The Ecological Stakes:**
- Vital greenspaces being converted to development
- Old-growth forests at risk
- Urban wildlife corridors fragmented
- Biodiversity loss accelerating

### Why Current Planning Falls Short

| Challenge | Impact |
|-----------|--------|
| **Siloed data** | Ecological data not integrated with zoning decisions |
| **Manual analysis** | Takes weeks to assess environmental impact |
| **Inaccessible insights** | Complex GIS analysis excludes community stakeholders |
| **Reactive approach** | Problems identified after development starts |

### Business Impact
- **Legal/environmental conflicts** — costly delays and lawsuits
- **Community opposition** — projects stalled by public outcry
- **Sustainability goal failures** — city missing climate commitments
- **Lost biodiversity** — irreversible ecosystem damage

---

## 🔧 Approach

### Solution Design
Build a **GenAI-powered planning assistant** that:
1. Ingests spatial, ecological, and planning data
2. Analyzes development proposals against environmental constraints
3. Generates site-specific recommendations with visualizations
4. Produces plain-language summaries for all stakeholders

### Design Philosophy

```
┌─────────────────────────────────────────────────────────────┐
│              BALANCE EQUATION                                │
│                                                              │
│     Housing Density  ←──  GenAI Analysis  ──→  Ecological   │
│     Optimization           Engine              Preservation  │
│                                                              │
│     • Affordable units     • Trade-off         • Greenspace │
│     • Zoning compliance      modeling          • Corridors  │
│     • Infrastructure       • Scenario sim      • Canopy     │
└─────────────────────────────────────────────────────────────┘
```

### Key Design Decisions

**1. RAG for Planning Documents**
- Ingest historical plans, zoning codes, and policies
- Enable natural language queries against decades of planning knowledge
- Chunking strategy: 512 tokens with 20% overlap for context preservation

**2. Multi-Stakeholder Outputs**
- Technical GIS layers for planners
- Plain-language summaries for community meetings
- "Green Score" ratings for quick decision-making

**3. AWS-Native Architecture**
- Serverless for cost efficiency
- Managed AI services (Bedrock) for reduced operational burden
- Scalable to handle city-wide data

---

## 🏗️ Architecture

### High-Level System Flow

```
┌───────────────────────────────────────────────────────────────────┐
│                     SEATTLE ECOPLANNER SYSTEM                      │
│                                                                    │
│  ┌─────────────┐    ┌─────────────────┐    ┌─────────────────┐   │
│  │   DATA      │    │    INGESTION    │    │    STORAGE      │   │
│  │   SOURCES   │───▶│    LAYER        │───▶│    LAYER        │   │
│  │             │    │                 │    │                 │   │
│  │ • Zoning    │    │ • ETL pipelines │    │ • Amazon S3     │   │
│  │ • Satellite │    │ • Chunking      │    │ • OpenSearch    │   │
│  │ • Ecology   │    │ • Embeddings    │    │   Serverless    │   │
│  │ • Feedback  │    │                 │    │                 │   │
│  └─────────────┘    └─────────────────┘    └────────┬────────┘   │
│                                                      │            │
│                                                      ▼            │
│  ┌─────────────┐    ┌─────────────────┐    ┌─────────────────┐   │
│  │   USER      │    │   APPLICATION   │    │    GenAI        │   │
│  │   LAYER     │◀───│   LAYER         │◀───│    ENGINE       │   │
│  │             │    │                 │    │                 │   │
│  │ • Planner   │    │ • REST APIs     │    │ • Bedrock       │   │
│  │   Dashboard │    │ • API Gateway   │    │ • Claude 3      │   │
│  │ • Community │    │ • Lambda        │    │ • Titan Embed   │   │
│  │   Portal    │    │                 │    │ • Knowledge     │   │
│  │ • GIS Tools │    │                 │    │   Bases         │   │
│  └─────────────┘    └─────────────────┘    └─────────────────┘   │
└───────────────────────────────────────────────────────────────────┘
```

### AWS Service Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        AWS ARCHITECTURE                              │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                     DATA INGESTION LAYER                      │   │
│  │  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐      │   │
│  │  │   S3    │   │  Glue   │   │ Lambda  │   │ Titan   │      │   │
│  │  │ Buckets │──▶│  ETL    │──▶│ Process │──▶│ Embed   │      │   │
│  │  └─────────┘   └─────────┘   └─────────┘   └────┬────┘      │   │
│  └──────────────────────────────────────────────────│───────────┘   │
│                                                      │               │
│  ┌──────────────────────────────────────────────────▼───────────┐   │
│  │                     KNOWLEDGE LAYER                           │   │
│  │  ┌─────────────────────────┐   ┌─────────────────────────┐   │   │
│  │  │   OpenSearch Serverless │   │   Bedrock Knowledge     │   │   │
│  │  │   (Vector Store)        │   │   Bases (RAG)           │   │   │
│  │  │   • Spatial vectors     │   │   • Planning docs       │   │   │
│  │  │   • 1024 dimensions     │   │   • Policy retrieval    │   │   │
│  │  └─────────────────────────┘   └─────────────────────────┘   │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                     ANALYSIS LAYER                            │   │
│  │  ┌─────────────────────────────────────────────────────────┐ │   │
│  │  │              Amazon Bedrock                              │ │   │
│  │  │  ┌───────────────┐  ┌───────────────┐  ┌─────────────┐  │ │   │
│  │  │  │ Claude 3      │  │ RetrieveAnd   │  │ Impact      │  │ │   │
│  │  │  │ Sonnet        │  │ Generate API  │  │ Scoring     │  │ │   │
│  │  │  │               │  │               │  │             │  │ │   │
│  │  │  │ • Analysis    │  │ • RAG queries │  │ • Green     │  │ │   │
│  │  │  │ • Summaries   │  │ • Context     │  │   Score     │  │ │   │
│  │  │  │ • Scenarios   │  │   injection   │  │ • Metrics   │  │ │   │
│  │  │  └───────────────┘  └───────────────┘  └─────────────┘  │ │   │
│  │  └─────────────────────────────────────────────────────────┘ │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                     PRESENTATION LAYER                        │   │
│  │  ┌─────────┐   ┌─────────┐   ┌─────────────┐   ┌─────────┐   │   │
│  │  │   API   │   │ Lambda  │   │  CloudFront │   │ Cognito │   │   │
│  │  │ Gateway │──▶│ Handler │──▶│     CDN     │──▶│  Auth   │   │   │
│  │  └─────────┘   └─────────┘   └─────────────┘   └─────────┘   │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                     SECURITY LAYER                            │   │
│  │  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐       │   │
│  │  │   IAM   │   │   KMS   │   │CloudTrail│   │  WAF    │       │   │
│  │  │ Roles   │   │ Encrypt │   │ Logging │   │ Protect │       │   │
│  │  └─────────┘   └─────────┘   └─────────┘   └─────────┘       │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

### Data Pipeline Detail

```
┌─────────────────────────────────────────────────────────────────────┐
│                     DATA PIPELINE FLOW                               │
│                                                                      │
│   1. INGESTION                                                       │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │  S3 Bucket Structure:                                        │   │
│   │  ├── /planning-docs/        (Historical plans, policies)    │   │
│   │  ├── /satellite-imagery/    (Landsat, aerial photos)        │   │
│   │  ├── /gis-layers/           (Zoning, parcels, ecology)      │   │
│   │  └── /community-feedback/   (Comments, surveys)             │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                              │                                       │
│                              ▼                                       │
│   2. PROCESSING                                                      │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │  Chunking Strategy:                                          │   │
│   │  • 512 tokens per chunk                                      │   │
│   │  • 20% overlap for context                                   │   │
│   │  • Metadata preserved (source, date, type)                   │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                              │                                       │
│                              ▼                                       │
│   3. EMBEDDING                                                       │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │  Amazon Titan Embeddings:                                    │   │
│   │  • 1024-dimensional vectors                                  │   │
│   │  • Optimized for semantic search                             │   │
│   │  • Batch processing via Lambda                               │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                              │                                       │
│                              ▼                                       │
│   4. INDEXING                                                        │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │  OpenSearch Serverless:                                      │   │
│   │  • Hybrid search (semantic + keyword)                        │   │
│   │  • Auto-scaling collections                                  │   │
│   │  • Real-time updates                                         │   │
│   └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 💻 Code Patterns

### Pattern 1: ArcGIS Integration
Bidirectional sync with existing GIS infrastructure:

```python
from arcgis.gis import GIS
from arcgis.features import FeatureLayer

def integrate_with_arcgis(bedrock_analysis_results):
    """Push AI analysis results to ArcGIS for visualization"""
    
    # Connect to ArcGIS Enterprise/Online
    gis = GIS(
        "https://your-arcgis-portal.com", 
        "username", 
        "api_key"
    )
    
    # Define feature layer schema for AI results
    properties = {
        "name": "AI_Ecological_Analysis",
        "spatialReference": {"wkid": 4326},  # WGS84
        "fields": [
            {"name": "green_score", "type": "esriFieldTypeDouble"},
            {"name": "ecological_impact", "type": "esriFieldTypeString"},
            {"name": "recommendation", "type": "esriFieldTypeString"},
            {"name": "confidence", "type": "esriFieldTypeDouble"}
        ]
    }
    
    # Create new feature layer with analysis results
    feature_layer = gis.content.create_feature_layer(properties)
    
    # Add features from Bedrock analysis
    feature_layer.edit_features(adds=bedrock_analysis_results)
    
    return feature_layer.url
```

### Pattern 2: QGIS Plugin Architecture
Desktop GIS integration for planners:

```python
from qgis.PyQt.QtWidgets import QDialog, QAction
from qgis.core import QgsVectorLayer, QgsProject
import requests

class SeattleEcoPlannerPlugin:
    """QGIS plugin for ecological analysis"""
    
    def __init__(self, iface):
        self.iface = iface
        self.bedrock_endpoint = "https://api.example.com/analyze"
    
    def initGui(self):
        """Create plugin UI elements"""
        self.action = QAction('EcoPlanner Analysis', self.iface.mainWindow())
        self.action.triggered.connect(self.run_analysis)
        self.iface.addPluginToMenu('Seattle EcoPlanner', self.action)
    
    def run_analysis(self):
        """Execute Bedrock analysis on selected area"""
        
        # Get selected geometry from QGIS
        selected_layer = self.iface.activeLayer()
        selected_features = selected_layer.selectedFeatures()
        
        for feature in selected_features:
            geometry = feature.geometry().asJson()
            
            # Call Bedrock API
            response = requests.post(
                self.bedrock_endpoint,
                json={"geometry": geometry, "analysis_type": "ecological_impact"}
            )
            
            analysis_results = response.json()
        
        # Create visualization layer
        layer = QgsVectorLayer('Polygon?crs=EPSG:4326', 'Ecological Analysis', 'memory')
        
        # Add fields and features
        provider = layer.dataProvider()
        provider.addAttributes([
            QgsField("green_score", QVariant.Double),
            QgsField("recommendation", QVariant.String)
        ])
        
        # Add to QGIS map canvas
        QgsProject.instance().addMapLayer(layer)
```

### Pattern 3: Bedrock Knowledge Base Query
RAG implementation for planning document retrieval:

```python
import boto3
import json

def query_planning_knowledge_base(query: str, site_context: dict) -> dict:
    """Query Bedrock Knowledge Base with RAG pattern"""
    
    bedrock_agent = boto3.client('bedrock-agent-runtime')
    
    # Construct enriched query with site context
    enriched_query = f"""
    Site Location: {site_context['coordinates']}
    Zoning: {site_context['zoning_code']}
    Proposed Use: {site_context['proposed_use']}
    
    Question: {query}
    """
    
    # Call RetrieveAndGenerate API
    response = bedrock_agent.retrieve_and_generate(
        input={
            'text': enriched_query
        },
        retrieveAndGenerateConfiguration={
            'type': 'KNOWLEDGE_BASE',
            'knowledgeBaseConfiguration': {
                'knowledgeBaseId': 'kb-seattle-planning-docs',
                'modelArn': 'arn:aws:bedrock:us-west-2::foundation-model/anthropic.claude-3-sonnet',
                'retrievalConfiguration': {
                    'vectorSearchConfiguration': {
                        'numberOfResults': 10,
                        'overrideSearchType': 'HYBRID'  # Semantic + keyword
                    }
                },
                'generationConfiguration': {
                    'promptTemplate': {
                        'textPromptTemplate': PLANNING_PROMPT_TEMPLATE
                    }
                }
            }
        }
    )
    
    return {
        'answer': response['output']['text'],
        'citations': response['citations'],
        'retrieved_docs': [c['retrievedReferences'] for c in response['citations']]
    }

# Prompt template for planning context
PLANNING_PROMPT_TEMPLATE = """
You are an expert urban planner analyzing development proposals for Seattle.
Use the following retrieved planning documents to answer the question.
Always cite specific policies, codes, or precedents when making recommendations.

Retrieved Documents:
$search_results$

Question: $query$

Provide:
1. Direct answer to the question
2. Relevant policy citations
3. Potential environmental considerations
4. Recommended next steps
"""
```

### Pattern 4: Green Score Calculation
Automated ecological impact assessment:

```python
import boto3
import json

def calculate_green_score(site_data: dict) -> dict:
    """Calculate ecological impact score for a development site"""
    
    bedrock = boto3.client('bedrock-runtime')
    
    # Structured prompt for consistent scoring
    scoring_prompt = f"""
    Analyze this development site and provide an ecological impact assessment.
    
    Site Data:
    - Location: {site_data['coordinates']}
    - Area: {site_data['area_sqft']} sq ft
    - Current Land Use: {site_data['current_use']}
    - Proposed Development: {site_data['proposed_development']}
    - Tree Canopy Coverage: {site_data['canopy_percent']}%
    - Distance to Wildlife Corridor: {site_data['corridor_distance_m']}m
    - Existing Greenspace: {site_data['greenspace_percent']}%
    
    Provide a JSON response with:
    {{
        "green_score": <0-100 score>,
        "impact_level": "low|medium|high|critical",
        "key_concerns": [<list of ecological concerns>],
        "mitigation_recommendations": [<list of recommendations>],
        "confidence": <0.0-1.0>
    }}
    """
    
    response = bedrock.invoke_model(
        modelId='anthropic.claude-3-sonnet-20240229-v1:0',
        body=json.dumps({
            "anthropic_version": "bedrock-2023-05-31",
            "max_tokens": 1024,
            "messages": [
                {"role": "user", "content": scoring_prompt}
            ]
        })
    )
    
    result = json.loads(response['body'].read())
    return json.loads(result['content'][0]['text'])
```

### Pattern 5: React Dashboard Component
Community-facing visualization:

```javascript
import React, { useState, useEffect } from 'react';
import Map, { Source, Layer } from 'react-map-gl';

const PlanningDashboard = () => {
  const [analysisResults, setAnalysisResults] = useState(null);
  const [selectedSite, setSelectedSite] = useState(null);
  
  // Fetch analysis from Bedrock API
  const analyzesite = async (siteId) => {
    const response = await fetch('/api/analyze-site', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ siteId })
    });
    const data = await response.json();
    setAnalysisResults(data);
  };
  
  // Color scale based on green score
  const getImpactColor = (score) => {
    if (score >= 80) return [34, 197, 94];    // Green - low impact
    if (score >= 60) return [250, 204, 21];   // Yellow - medium
    if (score >= 40) return [249, 115, 22];   // Orange - high
    return [239, 68, 68];                      // Red - critical
  };
  
  return (
    <div className="dashboard">
      <Map
        initialViewState={{
          latitude: 47.6062,
          longitude: -122.3321,
          zoom: 12
        }}
        mapStyle="mapbox://styles/mapbox/light-v11"
        onClick={(e) => setSelectedSite(e.features[0])}
      >
        {analysisResults && (
          <Source type="geojson" data={analysisResults}>
            <Layer
              id="eco-analysis"
              type="fill"
              paint={{
                'fill-color': [
                  'interpolate',
                  ['linear'],
                  ['get', 'green_score'],
                  0, '#ef4444',
                  50, '#f97316', 
                  75, '#facc15',
                  100, '#22c55e'
                ],
                'fill-opacity': 0.7
              }}
            />
          </Source>
        )}
      </Map>
      
      {selectedSite && (
        <SiteAnalysisPanel 
          site={selectedSite}
          analysis={analysisResults}
        />
      )}
    </div>
  );
};

const SiteAnalysisPanel = ({ site, analysis }) => (
  <div className="analysis-panel">
    <h2>Site Analysis</h2>
    <div className="green-score">
      <span className="score">{analysis.green_score}</span>
      <span className="label">Green Score</span>
    </div>
    <div className="recommendations">
      <h3>Recommendations</h3>
      <ul>
        {analysis.mitigation_recommendations.map((rec, i) => (
          <li key={i}>{rec}</li>
        ))}
      </ul>
    </div>
  </div>
);
```

### Pattern 6: API Gateway Configuration
Serverless REST API definition:

```yaml
# AWS SAM / CloudFormation template
openapi: 3.0.0
info:
  title: Seattle EcoPlanner API
  version: 1.0.0

paths:
  /analyze-site:
    post:
      summary: Analyze site ecological impact
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                siteId:
                  type: string
                geometry:
                  type: object
                analysisType:
                  type: string
                  enum: [ecological_impact, green_score, scenario_simulation]
      x-amazon-apigateway-integration:
        uri: !Sub "arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${AnalyzeSiteFunction.Arn}/invocations"
        passthroughBehavior: when_no_match
        httpMethod: POST
        type: aws_proxy
      
  /query-knowledge-base:
    post:
      summary: Query planning documents
      x-amazon-apigateway-integration:
        uri: !Sub "arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${QueryKBFunction.Arn}/invocations"
        httpMethod: POST
        type: aws_proxy
```

---

## 📊 Results

### Evaluation Criteria Assessment

| Criterion | Score | Implementation |
|-----------|-------|----------------|
| **Security** | ✅ Strong | IAM roles, KMS encryption, audit trails, GDPR/CCPA compliance |
| **Computational Efficiency** | ✅ High | Batch ETL, caching, serverless auto-scaling |
| **Agentic Automation** | ✅ Excellent | Autonomous analysis, proactive recommendations, scenario simulation |
| **Cost Efficiency** | ✅ Optimized | Pay-as-you-go, $50k-200k MVP estimate |
| **Response Time** | ✅ Fast | Sub-second for basic queries, caching for frequent requests |
| **Predictability** | ✅ High | Explainable AI, confidence scores, reproducible results |
| **Reusability** | ✅ Modular | Plug-and-play APIs, adaptable to other cities |

### Performance Metrics

| Metric | Target | Notes |
|--------|--------|-------|
| **Query Response** | <1 second | Basic queries with caching |
| **Impact Assessment Accuracy** | 95% | Against expert validation |
| **Spatial Data Processing** | Real-time | City-wide scale |
| **Concurrent Users** | 100+ | During community engagement sessions |

### Cost Estimation (Monthly)

| Component | Usage | Estimated Cost |
|-----------|-------|----------------|
| **Bedrock (Claude 3)** | 1.5M tokens | ~$15-30 |
| **OpenSearch Serverless** | 10 OCU | ~$700 |
| **S3 Storage** | 500GB | ~$12 |
| **S3 Requests** | 115K/month | ~$5 |
| **Lambda** | 1M invocations | ~$20 |
| **API Gateway** | 1M requests | ~$3.50 |
| **Total** | | **~$800-1000/month** |

### Model Evaluation Metrics

| Model Task | Metric | Target |
|------------|--------|--------|
| **Habitat Detection** | Precision/Recall | >90% |
| **Corridor Mapping** | IoU (Intersection over Union) | >85% |
| **Summary Quality** | BLEU/ROUGE | >0.7 |
| **User Satisfaction** | Rating | >4.5/5 |

### Sample Outputs

**Green Score Report:**
```
Site: 4521 Aurora Ave N
Proposed: 150-unit affordable housing

GREEN SCORE: 67/100 (Medium Impact)

Key Concerns:
• 3 mature Douglas Firs (>100 years) in development footprint
• Adjacent to Licton Springs wildlife corridor
• 42% existing tree canopy at risk

Recommendations:
1. Relocate building footprint 20m south to preserve old-growth trees
2. Install green roof (min 60% coverage) to maintain canopy function
3. Create wildlife passage under parking structure
4. Native plantings buffer along corridor edge

Policy Citations:
• Seattle Municipal Code 25.11 (Tree Protection)
• Comprehensive Plan Policy EN 1.2 (Wildlife Corridors)
```

---

## 🔑 Key Takeaways

### Architecture Patterns to Reuse

| Pattern | Use Case |
|---------|----------|
| **Bedrock Knowledge Bases** | Any RAG application with document retrieval |
| **OpenSearch Serverless** | Hybrid semantic + keyword search |
| **GIS Integration** | Spatial AI applications |
| **Multi-stakeholder outputs** | Systems serving technical and non-technical users |
| **Green scoring models** | Quantifiable impact assessments |

### AWS Best Practices Demonstrated

1. **Serverless-first** — Lambda + API Gateway for cost efficiency
2. **Managed AI services** — Bedrock reduces operational burden
3. **Hybrid search** — Combine vector + keyword for better retrieval
4. **Security layers** — IAM, KMS, WAF, CloudTrail for compliance
5. **Modular design** — Each layer independently scalable

### Scalability Considerations

- **Horizontal scaling** — OpenSearch Serverless auto-scales
- **Caching layer** — CloudFront + ElastiCache for repeat queries
- **Batch processing** — Glue for large geospatial datasets
- **Multi-region** — Disaster recovery and latency optimization

### Go-to-Market Strategy

1. **Pilot** — Seattle city planning department
2. **Partners** — Forterra, Seattle Parks Foundation, UW
3. **Integration** — ArcGIS, QGIS, municipal dashboards
4. **Community** — Workshops and feedback sessions
5. **Scale** — Template for other cities (Portland, Denver, etc.)

---

## 📁 Project Structure

```
seattle-ecoplanner/
├── infrastructure/
│   ├── template.yaml          # SAM/CloudFormation
│   ├── bedrock-kb-setup.py    # Knowledge Base configuration
│   └── opensearch-setup.py    # Vector store setup
├── lambda/
│   ├── analyze_site/          # Ecological analysis function
│   ├── query_kb/              # Knowledge base query function
│   └── calculate_score/       # Green score calculation
├── frontend/
│   ├── src/
│   │   ├── components/        # React components
│   │   └── hooks/             # Custom hooks for Bedrock
│   └── public/
├── plugins/
│   ├── qgis/                  # QGIS plugin
│   └── arcgis/                # ArcGIS integration
├── data/
│   └── sample/                # Sample planning documents
└── docs/
    └── architecture/          # Diagrams (draw.io)
```

---

## 🛠️ Technologies Used

| Category | Technology |
|----------|------------|
| **GenAI** | Amazon Bedrock (Claude 3 Sonnet) |
| **Embeddings** | Amazon Titan Embeddings |
| **RAG** | Bedrock Knowledge Bases |
| **Vector Store** | OpenSearch Serverless |
| **Storage** | Amazon S3 |
| **Compute** | AWS Lambda |
| **API** | Amazon API Gateway |
| **Auth** | Amazon Cognito |
| **Frontend** | React, Mapbox GL |
| **GIS Integration** | ArcGIS, QGIS |
| **Security** | IAM, KMS, WAF, CloudTrail |

---

## 📚 Data Sources

### Planning Documents
- [Seattle SDCI](https://www.seattle.gov/sdci) — Zoning maps, land use codes
- [Seattle Municipal Archives](https://www.seattle.gov/city-archives) — Historical records

### Satellite Imagery
- [USGS Earth Explorer](https://earthexplorer.usgs.gov/) — Landsat imagery
- [NASA Earthdata](https://earthdata.nasa.gov/) — Multi-satellite datasets

### GIS Layers
- [Seattle GIS Program](https://www.seattle.gov/city-gis) — City boundaries, infrastructure
- [King County GIS](https://kingcounty.gov/services/gis.aspx) — Regional environmental data

### Community Feedback
- [Seattle Open Data Portal](https://data.seattle.gov/) — Public datasets, surveys
- [Seattle In Progress](https://www.seattleinprogress.com/) — Development feedback

---

*Case study demonstrating enterprise-scale GenAI architecture for urban planning*
