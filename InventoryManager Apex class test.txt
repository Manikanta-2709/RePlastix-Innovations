@isTest

public class InventoryManagerTest {    

    @testSetup

    static void setupTestData() {

        // Create a product with initial stock of 50

        Re_Plastic_Innovations_Recycled_Product__c product = new Re_Plastic_Innovations_Recycled_Product__c(

            Stock_Level__c = 50,

            Threshold__c = 10

        );

        insert product;

        // Create orders

        List<Re_Plastic_Innovations_Order__c> orders = new List<Re_Plastic_Innovations_Order__c>{

            new Re_Plastic_Innovations_Order__c(Recycled_Product__c = product.Id, Quantity__c = 20), // Enough stock

            new Re_Plastic_Innovations_Order__c(Recycled_Product__c = product.Id, Quantity__c = 40)  // Not enough stock

        };

        insert orders;

    }

    @isTest

    static void testProcessOrderStock() {

        List<Re_Plastic_Innovations_Order__c> orders = [SELECT Id, Recycled_Product__c, Quantity__c FROM Re_Plastic_Innovations_Order__c];

        

        Test.startTest();

        InventoryManager.processOrderStock(orders);

        Test.stopTest();

        // Check updated stock

        Re_Plastic_Innovations_Recycled_Product__c product = [SELECT Stock_Level__c FROM Re_Plastic_Innovations_Recycled_Product__c];

        System.assertEquals(10, product.Stock_Level__c, 'Stock should reduce correctly');

        // Check restock request (Only 1 should be created)

        List<Re_Plastic_Innovations_Restock_Request__c> restockRequests = [SELECT Id FROM Re_Plastic_Innovations_Restock_Request__c WHERE Status__c = 'Pending'];

        System.assertEquals(2, restockRequests.size(), 'Only one restock request should be created');

    }

   @isTest

static void testProcessRestockApproval() {

    // Fetch restock requests and approve them

    List<Re_Plastic_Innovations_Restock_Request__c> restocks = [SELECT Id, Product__c, Requested_Quantity__c FROM Re_Plastic_Innovations_Restock_Request__c];

    for (Re_Plastic_Innovations_Restock_Request__c req : restocks) {

        req.Status__c = 'Approved';

    }

    update restocks;

    // Calculate expected stock level before processing restock

    Re_Plastic_Innovations_Recycled_Product__c productBefore = [SELECT Stock_Level__c FROM Re_Plastic_Innovations_Recycled_Product__c];

    Decimal initialStock = productBefore.Stock_Level__c;

    // Calculate total requested quantity for approved restocks

    Decimal totalRestockQuantity = 0;

    for (Re_Plastic_Innovations_Restock_Request__c req : restocks) {

        totalRestockQuantity += req.Requested_Quantity__c;

    }

    // Process restock approvals

    Test.startTest();

    InventoryManager.processRestockApproval(restocks);

    Test.stopTest();

    // Fetch updated stock

    Re_Plastic_Innovations_Recycled_Product__c productAfter = [SELECT Stock_Level__c FROM Re_Plastic_Innovations_Recycled_Product__c];

    

    // Expected stock = Initial stock + Total approved restock quantity

    Decimal expectedStock = initialStock + totalRestockQuantity;

    

    System.assertEquals(expectedStock, productAfter.Stock_Level__c, 'Stock should increase correctly after restock approval');

}

}