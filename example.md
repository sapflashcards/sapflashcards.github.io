```mermaid
flowchart TD
    %% Define pools/lanes
    subgraph CustomerPool ["Customer"]
        CS_Start([Customer])
    end
    
    subgraph SAPSystem ["SAP System"]
        subgraph SalesDept ["Sales Department"]
            SE_Start(["Order Inquiry Received"])
            SO_Quote["Create Sales Quotation\n(VA21)"]
            SO_Receive["Receive Customer Purchase Order"]
            SO_Create["Create Sales Order\n(VA01)"]
            SO_ATP["Perform Availability Check (ATP)"]
            SO_CheckGateway{{"Stock & Credit OK?"}}
            SO_CreditBlock["Handle Credit Block"]
            SO_StockShortage["Handle Stock Shortage"]
        end
        
        subgraph Warehouse ["Warehouse/Logistics"]
            WH_Delivery["Create Outbound Delivery\n(VL01N)"]
            WH_Picking["Perform Picking"]
            WH_Packing["Perform Packing\n(Optional)"]
            WH_PGI["Post Goods Issue (PGI)\n(VL02N)"]
            WH_Ship["Ship Goods to Customer"]
        end
        
        subgraph Finance ["Finance/Billing Department"]
            FI_CreateInvoice["Create Billing Document\n(VF01)"]
            FI_SendInvoice["Send Invoice to Customer"]
            FI_WaitPayment["Wait for Payment Due Date"]
            FI_ReceivePayment["Receive Customer Payment"]
            FI_PostPayment["Post Incoming Payment & Clear A/R Item\n(F-28)"]
            FI_End["Payment Received and Cleared\nO2C Complete"]
        end
    end
    
    %% Define sequence flows
    SE_Start --> SO_Quote
    SO_Quote --> SO_Receive
    SO_Receive --> SO_Create
    SO_Create --> SO_ATP
    SO_ATP --> SO_CheckGateway
    
    SO_CheckGateway -->|"Yes"| WH_Delivery
    SO_CheckGateway -->|"No - Credit Issue"| SO_CreditBlock
    SO_CreditBlock --> SO_CheckGateway
    SO_CheckGateway -->|"No - Stock Issue"| SO_StockShortage
    SO_StockShortage --> SO_CheckGateway
    
    WH_Delivery --> WH_Picking
    WH_Picking --> WH_Packing
    WH_Packing --> WH_PGI
    WH_PGI --> WH_Ship
    WH_Ship --> FI_CreateInvoice
    FI_CreateInvoice --> FI_SendInvoice
    FI_SendInvoice --> FI_WaitPayment
    FI_WaitPayment --> FI_ReceivePayment
    FI_ReceivePayment --> FI_PostPayment
    FI_PostPayment --> FI_End
    
    %% Define message flows
    CS_Start -->|"Purchase Order/Inquiry"| SE_Start
    WH_Ship -.->|"Advance Shipping Notification"| CustomerPool
    FI_SendInvoice -.->|"Invoice Document"| CustomerPool
    CustomerPool -.->|"Payment Advice/Bank Transfer"| FI_ReceivePayment

    %% Styling
    classDef start fill:#9D9,stroke:#696,stroke-width:2px;
    classDef end fill:#F99,stroke:#966,stroke-width:2px;
    classDef task fill:#89CFF0,stroke:#0078D7,stroke-width:1px;
    classDef gateway fill:#ffffc0,stroke:#ccc,stroke-width:1px;
    classDef timer fill:#fbd2c9,stroke:#8f4236,stroke-width:1px;
    
    class SE_Start start;
    class FI_End end;
    class SO_Quote,SO_Receive,SO_Create,SO_ATP,SO_CreditBlock,SO_StockShortage,WH_Delivery,WH_Picking,WH_Packing,WH_PGI,WH_Ship,FI_CreateInvoice,FI_SendInvoice,FI_ReceivePayment,FI_PostPayment task;
    class SO_CheckGateway gateway;
    class FI_WaitPayment timer;
```
