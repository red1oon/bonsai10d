# Bonsai 10D: Integrated BIM-to-ERP Vision
## Actual Working POC - based on active IFCs of TERMINAL 1/2 Jetty Complex Project, Malaysia exported from Autodesk Revit

### Understanding BIM Dimensions

Building Information Modeling has evolved beyond simple 3D geometry. The industry recognizes these dimensional progressions:

**3D** - Geometric model (length, width, height)  
**4D** - Time/scheduling (construction sequencing)  
**5D** - Cost estimation and budgeting  
**6D** - Facility management and operations  
**7D** - Sustainability and energy analysis  

Some practitioners extend to **8D** (safety planning) and beyond, though these aren't universally standardized. The concept of "10D" represents the integration endpoint: **complete digital continuity from design through construction to enterprise resource planning.**

### What Bonsai Currently Provides

Bonsai (BlenderBIM) delivers production-ready capabilities across multiple dimensions:

#### 2D - Documentation and Drawings
- **`drawing` module**: Generate construction document sets, sections, elevations
- **`context` module**: Annotations, dimensions, text notes on drawings  
- Sheet management with title blocks and revision tracking
- SVG/PDF export for distribution to contractors
- Supports multi-scale documentation workflows

#### 3D - Geometric Modeling
- **`geometry` module**: Parametric IFC geometry creation and editing
- **`model` module**: Direct modeling tools for walls, slabs, columns, beams
- **`spatial` module**: Building hierarchy (Site → Building → Storey → Space)
- **`material` module**: Material assignments with properties
- **`style` module**: Surface styles, presentation layers
- Native IFC authoring—not translation from proprietary formats

#### 4D - Time and Scheduling
- **`sequence` module**: Work schedules, task hierarchies, Gantt visualization
- Task-to-element mapping for visual construction sequencing
- Predecessor/successor relationships with lag times
- Calendar-based scheduling with work hours configuration
- 4D animation playback in Blender viewport

#### 5D - Cost Estimation
- **`cost` module**: Cost schedules, cost items, budget breakdowns
- **`qto` module**: Automated quantity take-offs from IFC geometry
- **`resource` module**: Labor, equipment, material resource assignment
- Unit cost assignment with currency support
- Cost value inheritance and aggregation through building hierarchy

#### 6D - Facility Management
- **`fm` module**: Asset information for operations and maintenance
- Property sets (Psets) for equipment specifications
- **`document` module**: Link O&M manuals, warranties, submittals
- Space management and zone definitions
- COBie export for handover to facility management systems

### The 10D Vision: BIM-to-ERP Integration

For our engineering firm, "10D" means closing the loop between BIM models and iDempiere ERP. This is not marketing—it's about eliminating manual data re-entry and keeping project data synchronized.

#### Practical Integration Points

```python
# Example: Cost data flows from BIM to ERP
from bonsai.bim import tool
import ifcopenshell.api

# 1. Extract quantity take-offs from IFC
ifc_file = tool.Ifc.get()
cost_schedule = ifc_file.by_type("IfcCostSchedule")[0]

for cost_item in ifc_file.by_type("IfcCostItem"):
    # Extract quantities
    quantities = ifcopenshell.util.element.get_psets(cost_item)
    
    # 2. Map to iDempiere product/material master
    material_id = cost_item.Name  # Links to M_Product
    quantity = quantities.get("Qto_BaseQuantities", {}).get("Length", 0)
    unit_cost = cost_item.CostValues[0].AppliedValue if cost_item.CostValues else 0
    
    # 3. Generate procurement requisition in ERP
    create_purchase_requisition(
        material_id=material_id,
        quantity=quantity,
        project_phase=get_4d_schedule_phase(cost_item),
        budget_line=cost_item.GlobalId
    )
```

#### Two-Way Data Flow

**BIM → ERP:**
- Quantity take-offs → Material requirements planning
- 4D schedule → Procurement timing
- Cost estimates → Budget approval workflow
- As-built updates → Asset register

**ERP → BIM:**
- Actual costs → Budget variance analysis
- Delivery schedules → 4D timeline adjustments
- Material specifications → Element property updates
- Change orders → Model revision triggers

### Technical Architecture (High-Level)

```
┌──────────────────────────────────────────────────────────┐
│ BONSAI (BlenderBIM)                                      │
│                                                          │
│  [3D Model] ←→ [4D Sequence] ←→ [5D Cost] ←→ [6D FM]   │
│       ↓              ↓              ↓           ↓        │
│  [IFC File] ──────── [Data Layer] ─────────────┘        │
└────────────────────────┬─────────────────────────────────┘
                         │
                  [Integration Layer]
                  - REST API adapter
                  - IFC property mapping
                  - Change event detection
                         │
┌────────────────────────┴─────────────────────────────────┐
│ iDEMPIERE ERP                                            │
│                                                          │
│  [Project Mgmt] ←→ [Procurement] ←→ [Accounting]       │
│  [Work Orders]  ←→ [Inventory]   ←→ [Document Mgmt]    │
└──────────────────────────────────────────────────────────┘
```

### Existing Bonsai Modules We Build Upon

No reinvention—we leverage what the community already provides:

**Coordination & Analysis:**
- **`clash`**: Geometric clash detection with BCF output
- **`system`**: MEP system definitions (electrical, mechanical, plumbing)
- **`structural`**: Structural analysis integration
- **`bsdd`**: BuildingSMART Data Dictionary for standardized properties

**Data Management:**
- **`pset`**: Property set management (custom attributes)
- **`classification`**: Link to classification systems (Uniclass, Omniclass)
- **`library`**: Reusable element type libraries
- **`owner`**: Stakeholder and organization assignments

**Documentation:**
- **`bcf`**: BIM Collaboration Format for issue tracking
- **`csv`**: Import/export data to spreadsheets
- **`document`**: External document references

**Quality Control:**
- **`tester`**: IFC validation against buildingSMART standards
- **`diff`**: Compare IFC model versions
- **`ifcgit`**: Version control integration

### What We're Adding

**New `erp_bridge` module** (planned):
- IFC-to-ERP entity mapping configuration
- REST API client for iDempiere
- Bi-directional sync orchestration
- Change detection and conflict resolution
- Audit trail for data transfers

**Enhanced MEP capabilities** (in progress):
- Conduit routing optimization
- Clash detection with spatial indexing
- Multi-discipline coordination for jetty infrastructure

### Real-World Benefits for Terminal 1/2

1. **Accurate budgeting**: BIM quantities automatically update ERP cost centers
2. **Procurement automation**: Material orders generated from 4D schedule milestones
3. **Progress tracking**: As-built updates flow from field tablets through BIM to ERP
4. **Document control**: All RFIs, submittals, and change orders linked by GlobalId
5. **Handover efficiency**: Asset data pre-populated in ERP for operations phase

### Not Overselling

This integration requires:
- Manual mapping of IFC entities to ERP entities (one-time setup per project type)
- Clear data governance policies
- Training for both BIM and ERP users
- Fallback processes when systems are out of sync

The "10D" label is shorthand for this vision, not a magic solution. The work is in the integration logic, not in inventing new dimensions.

### Current Status

- ✅ 3D-6D: Production-ready in Bonsai
- 🚧 7D: Sustainability analysis via external plugins
- 📋 **8D (Safety Management)**: Health and safety planning during design and construction—integrates with ERP for safety equipment procurement, incident tracking, and compliance documentation
- 📋 **9D (Lean Construction)**: Process optimization and waste reduction—ERP integration enables just-in-time procurement, warehouse inventory tracking, material deployment scheduling, labour productivity costing, and resource utilization monitoring across the production cycle
- 📋 **10D (Construction Industrialization)**: Prefabrication and off-site manufacturing coordination—ERP integration provides vendor marketplace management, pricing optimization, financial consolidation across all dimensions, and regulatory reporting/documentation workflows from design through operations

Status: 8D-10D integration layer under development for Terminal 1/2 pilot project

### References

- BuildingSMART IFC4.3 standard for infrastructure
- Bonsai documentation: https://docs.bonsaibim.org/
- iDempiere ERP: https://www.idempiere.org/
- Industry BIM dimension definitions: See ISO 19650 series

---

**Bottom Line**: Bonsai 10D isn't about adding more dimensions to BIM—it's about making existing BIM data (3D through 6D) seamlessly flow into enterprise systems like iDempiere, eliminating duplicate data entry and keeping project information synchronized from design through operations.

**Author**:
Redhuan D. Oon (red1)

**Developers**:
Naquib Danial Oon

**Company Reference**:
IKTHISAS INGENIURS SDN BHD
TAMAN MELAWATI
MALAYSIA

**LICENSING OF CODE**
Committed to FLOSS, compliance with BONSAI's GPL v3.0 or later

**The Red1 Three Line Mantra during the 2006 turnaround fork**
- Information is Free, You have to Know
- People are Not, You have to Pay
- Contributors are Priceless, You have to Be

**4th Mantra**
- AI is here, leap further
