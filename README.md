# EAM WSHub Proxy Client
This project contains the Maven configuration that generates JAX-WS aisws aisws from EAM WSDLs,
using the JAX-WS Maven Plugin.

The API of the generated JAX-WS aisws is quite verbose. This is why we have created another project, 
[EAM WSHub Core](https://github.com/cern-eam/eam-wshub-core), that simplify the way you can call EAM from Java.

Only the most commonly used EAM WSDLs are currently taken into consideration. This list of WSDLs is defined inside pom.xml and can be extended if required.

**This is a low-level library included in [EAM WSHub Core](https://github.com/cern-eam/eam-wshub-core) 
and is not recommended to be used directly. Instead, the proposed way to call EAM web services is:**
 - **from Java: [EAM WSHub Core](https://github.com/cern-eam/eam-wshub-core)**
 - **from other languages: [EAM WSHub](https://github.com/cern-eam/eam-wshub)**

## Installation
The generated JAX-WS aisws is published as an artifact in Maven Central:
```
<dependency>
    <groupId>ch.cern.eam</groupId>
    <artifactId>eam-wshub-proxyclient</artifactId>
    <version>11.4-1</version>
</dependency>
```

## Usage
The following piece of code shows how to update the description of a work order with EAM WSHub Proxy Client:
```java
import net.datastream.schemas.mp_entities.workorder_001.WorkOrder;
import net.datastream.schemas.mp_fields.ORGANIZATIONID_Type;
import net.datastream.schemas.mp_fields.WOID_Type;
import net.datastream.schemas.mp_functions.mp0024_001.MP0024_GetWorkOrder_001;
import net.datastream.schemas.mp_functions.mp0025_001.MP0025_SyncWorkOrder_001;
import net.datastream.schemas.mp_results.mp0024_001.MP0024_GetWorkOrder_001_Result;
import net.datastream.wsdls.eamws.EAMWebServicesPT;
import org.xmlsoap.schemas.ws._2002._04.secext.*;
import javax.xml.namespace.QName;
import javax.xml.ws.BindingProvider;
import javax.xml.ws.Service;

public class UpdateWorkOrderExample {
    
    public static void main(String[] args) {
        
        String eamEndpointUrl = "{YOUR-ENDPOINT-URL}";
        String organizationCode = "{YOUR-ORGANIZATION-CODE}";
        String tenant = "{YOUR-TENANT}";
        String username = "{YOUR-USERNAME}";
        String password = "{YOUR-PASSWORD}";
        String workOrderNumber = "{YOUR-WORKORDER-NUMBER}";
        String sessionScenario = "TERMINATE";
        
        // Get EAMWebServicesPT instance
        Service service = Service.create(new QName("eamws"));
        EAMWebServicesPT eamWebServicesToolkitClient = service.getPort(EAMWebServicesPT.class);
        ((BindingProvider) eamWebServicesToolkitClient).getRequestContext().put(BindingProvider.ENDPOINT_ADDRESS_PROPERTY, eamEndpointUrl);
        
        // Creation of security header
        ObjectFactory of = new ObjectFactory();
        Security securityHeader = of.createSecurity();
        Username un = of.createUsername();
        un.setValue(username.toUpperCase());
        Password pass = of.createPassword();
        pass.setValue(password);
        UsernameToken unt = of.createUsernameToken();
        unt.setPassword(pass);
        unt.setUsername(un);
        securityHeader.getAny().add(unt);
        
        // Creation of MessageConfig element that can be added to the request to tell the
        // server to omit the InformationAlert and WarningAlert elements from the response.
        MessageConfigType messageConfigType = new MessageConfigType();
        MessageItemConfigType returnAlertsMessageItemConfigType = new MessageItemConfigType();
        returnAlertsMessageItemConfigType.setName("returnAlerts");
        returnAlertsMessageItemConfigType.setValue("false");
        messageConfigType.getConfigItem().add(returnAlertsMessageItemConfigType);
        
        // Creation of GetWorkOrder object
        MP0024_GetWorkOrder_001 getWorkOrderRequest = new MP0024_GetWorkOrder_001();
        WOID_Type woid = new WOID_Type();
        woid.setJOBNUM(workOrderNumber);
        ORGANIZATIONID_Type org = new ORGANIZATIONID_Type();
        org.setORGANIZATIONCODE(organizationCode);
        woid.setORGANIZATIONID(org);
        getWorkOrderRequest.setWORKORDERID(woid);
        
        // Fetch workorder
        MP0024_GetWorkOrder_001_Result getWorkOrderResult = eamWebServicesToolkitClient.getWorkOrderOp(
                getWorkOrderRequest,
                organizationCode,
                securityHeader,
                sessionScenario,
                null,
                messageConfigType,
                tenant
        );
        WorkOrder workOrder = getWorkOrderResult.getResultData().getWorkOrder();
        
        // Let's update the description
        workOrder.getWORKORDERID().setDESCRIPTION("New workOrder description");
        
        // And synchronize the workorder in EAM
        MP0025_SyncWorkOrder_001 syncWO = new MP0025_SyncWorkOrder_001();
        syncWO.setWorkOrder(workOrder);
        eamWebServicesToolkitClient.syncWorkOrderOp(
                syncWO,
                organizationCode,
                securityHeader,
                sessionScenario,
                null,
                messageConfigType,
                tenant);
        
    }
}
```

## License
This software is published under the GNU General Public License v3.0 or later.
