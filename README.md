#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>


typedef struct Product {
    int id;
    char name[50];
    int quantity;
    float price;
    char category[30];
} Product;

typedef struct CartItem {
    Product product;
    int quantity;
    struct CartItem *next;
} CartItem;

typedef struct Customer {
    char name[50];
    char address[100];
    char paymentMethod[20];
    CartItem *cart;
} Customer;

typedef struct Order {
    int orderId;
    char customerName[50];
    char customerAddress[100];
    char paymentMethod[20];
    Product *products;
    int productCount;
    char status[20];
    struct Order *next;
} Order;

typedef struct DeliveryStaff {
    int staffId;
    char name[50];
    struct DeliveryStaff *next;
} DeliveryStaff;

typedef struct Delivery {
    int deliveryId;
    Order *order;
    char currentLocation[50];
    struct Delivery *prev, *next;
} Delivery;


Order *orderQueue = NULL;
Product availableProducts[100];
int productCount = 0;
DeliveryStaff *staffStack = NULL;
Delivery *deliveryList = NULL;
Customer currentCustomer;


void adminLogin();
void customerLogin();
void addProduct();
void createOrder();
void assignDelivery();
void trackDelivery();
void generateReport();
void initializeSampleData();


void searchProducts();
void addToCart();
void viewCart();
void enterAddress();
void selectPayment();
void confirmOrder();
void cancelOrder();

int main() {

    initializeSampleData();

    int choice;
    while (1) {
        printf("\n===== Delivery Management System =====");
        printf("\n1. Admin Login");
        printf("\n2. Customer Login");
        printf("\n3. Exit");
        printf("\nEnter choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1: adminLogin(); break;
            case 2: customerLogin(); break;
            case 3: exit(0);
            default: printf("Invalid choice!\n");
        }
    }
    return 0;
}


void initializeSampleData() {

    availableProducts[0] = (Product){101, "Laptop", 10, 999.99, "Electronics"};
    availableProducts[1] = (Product){102, "Smartphone", 20, 699.99, "Electronics"};
    availableProducts[2] = (Product){103, "Headphones", 15, 149.99, "Electronics"};
    availableProducts[3] = (Product){201, "T-Shirt", 50, 19.99, "Clothing"};
    availableProducts[4] = (Product){202, "Jeans", 30, 49.99, "Clothing"};
    productCount = 5;


    DeliveryStaff *staff1 = (DeliveryStaff *)malloc(sizeof(DeliveryStaff));
    staff1->staffId = 501;
    strcpy(staff1->name, "John Doe");
    staff1->next = staffStack;
    staffStack = staff1;

    DeliveryStaff *staff2 = (DeliveryStaff *)malloc(sizeof(DeliveryStaff));
    staff2->staffId = 502;
    strcpy(staff2->name, "Jane Smith");
    staff2->next = staffStack;
    staffStack = staff2;

    printf("Sample data loaded (%d products, 2 staff).\n", productCount);
}


void adminLogin() {
    char username[20], password[20];
    printf("Admin Username: "); scanf("%s", username);
    printf("Admin Password: "); scanf("%s", password);

    if (strcmp(username, "amma") == 0 && strcmp(password, "amma123") == 0) {
        printf("Login successful!\n");
        int choice;
        while (1) {
            printf("\n===== Admin Panel =====");
            printf("\n1. Add Product");
            printf("\n2. View All Orders");
            printf("\n3. Assign Delivery");
            printf("\n4. Generate Report");
            printf("\n5. Logout");
            printf("\nEnter choice: ");
            scanf("%d", &choice);

            switch (choice) {
                case 1: addProduct(); break;
                case 2: generateReport(); break;
                case 3: assignDelivery(); break;
                case 4: generateReport(); break;
                case 5: return;
                default: printf("Invalid choice!\n");
            }
        }
    } else {
        printf("Login failed!\n");
    }
}

void addProduct() {
    printf("\n===== Add New Product =====p");
    printf("\nEnter Product ID: "); scanf("%d", &availableProducts[productCount].id);
    printf("Enter Product Name: "); scanf("%s", availableProducts[productCount].name);
    printf("Enter Category: "); scanf("%s", availableProducts[productCount].category);
    printf("Enter Quantity: "); scanf("%d", &availableProducts[productCount].quantity);
    printf("Enter Price: "); scanf("%f", &availableProducts[productCount].price);
    productCount++;
    printf("Product added successfully!\n");
}


void customerLogin() {
    printf("\nEnter your name: ");
    scanf("%s", currentCustomer.name);
    currentCustomer.cart = NULL;

    int choice;
    while (1) {
        printf("\n===== Customer Menu ======");
        printf("\n1. Search Products");
        printf("\n2. Add to Cart");
        printf("\n3. View Cart");
        printf("\n4. Enter Delivery Address");
        printf("\n5. Select Payment Method");
        printf("\n6. Confirm Order");
        printf("\n7. Track Order");
        printf("\n8. Cancel Order");
        printf("\n9. Exit");
        printf("\nEnter choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1: searchProducts(); break;
            case 2: addToCart(); break;
            case 3: viewCart(); break;
            case 4: enterAddress(); break;
            case 5: selectPayment(); break;
            case 6: confirmOrder(); break;
            case 7: trackDelivery(); break;
            case 8: cancelOrder(); break;
            case 9: return;
            default: printf("Invalid choice!\n");
        }
    }
}

void searchProducts() {
    char query[50];
    printf("\nEnter product name or category to search: ");
    scanf("%s", query);

    printf("\n===== Search Results =====");
    int found = 0;
    for (int i = 0; i < productCount; i++) {
        if (strstr(availableProducts[i].name, query) != NULL ||
            strstr(availableProducts[i].category, query) != NULL) {
            printf("\nID: %d | Name: %s | Category: %s", availableProducts[i].id,
                   availableProducts[i].name, availableProducts[i].category);
            printf("\nPrice: $%.2f | Stock: %d", availableProducts[i].price,
                   availableProducts[i].quantity);
            printf("\n--------------------------------");
            found = 1;
        }
    }
    if (!found) printf("\nNo products found matching '%s'", query);
}

void addToCart() {
    int id, qty;
    printf("\nEnter Product ID to add: ");
    scanf("%d", &id);
    printf("Enter Quantity: ");
    scanf("%d", &qty);

    Product *product = NULL;
    for (int i = 0; i < productCount; i++) {
        if (availableProducts[i].id == id) {
            product = &availableProducts[i];
            break;
        }
    }

    if (product == NULL) {
        printf("Product not found!\n");
        return;
    }

    if (qty > product->quantity) {
        printf("Sorry stock out!!! Only %d available in stock!\n", product->quantity);
        return;
    }

    CartItem *newItem = (CartItem *)malloc(sizeof(CartItem));
    newItem->product = *product;
    newItem->quantity = qty;
    newItem->next = currentCustomer.cart;
    currentCustomer.cart = newItem;

    printf("Added %d x %s to your cart!\n", qty, product->name);
}

void viewCart() {
    if (currentCustomer.cart == NULL) {
        printf("\nYour cart is empty!\n");
        return;
    }printf("\n===== Your Shopping Cart =====");
    CartItem *item = currentCustomer.cart;
    float total = 0;
    while (item != NULL) {
        printf("\n%s (Qty: %d) - $%.2f", item->product.name,
               item->quantity, item->product.price * item->quantity);
        total += item->product.price * item->quantity;
        item = item->next;
    }
    printf("\n-----------------------------");
    printf("\nTotal: $%.2f\n", total);
}

void enterAddress() {
    printf("\nEnter your delivery address: ");
    scanf(" %[^\n]", currentCustomer.address);
    printf("Address saved!\n");
}

void selectPayment() {
    printf("\n===== Payment Methods =====");
    printf("\n1. Cash on Delivery");
    printf("\n2. Credit/Debit Card");
    printf("\n3. Mobile Payment");
    printf("\nEnter your choice: ");
    int choice;
    scanf("%d", &choice);

    switch (choice) {
        case 1: strcpy(currentCustomer.paymentMethod, "Cash"); break;
        case 2: strcpy(currentCustomer.paymentMethod, "Card"); break;
        case 3: strcpy(currentCustomer.paymentMethod, "Mobile"); break;
        default: printf("Invalid choice!\n"); return;
    }
    printf("Payment method set to: %s\n", currentCustomer.paymentMethod);
}

void confirmOrder() {
    if (currentCustomer.cart == NULL) {
        printf("\nYour cart is empty!\n");
        return;
    }
    if (strlen(currentCustomer.address) == 0) {
        printf("\nPlease enter delivery address first!\n");
        return;
    }
    if (strlen(currentCustomer.paymentMethod) == 0) {
        printf("\nPlease select payment method first!\n");
        return;
    }


    Order *newOrder = (Order *)malloc(sizeof(Order));
    newOrder->orderId = 1000 + rand() % 9000;
    strcpy(newOrder->customerName, currentCustomer.name);
    strcpy(newOrder->customerAddress, currentCustomer.address);
    strcpy(newOrder->paymentMethod, currentCustomer.paymentMethod);
    strcpy(newOrder->status, "Queue");


    int itemCount = 0;
    CartItem *temp = currentCustomer.cart;
    while (temp != NULL) {
        itemCount++;
        temp = temp->next;
    }


    newOrder->products = (Product *)malloc(itemCount * sizeof(Product));
    newOrder->productCount = 0;


    temp = currentCustomer.cart;
    while (temp != NULL) {
        newOrder->products[newOrder->productCount++] = temp->product;

        for (int i = 0; i < productCount; i++) {
            if (availableProducts[i].id == temp->product.id) {
                availableProducts[i].quantity -= temp->quantity;
                break;
            }
        }
        temp = temp->next;
    }


    newOrder->next = NULL;
    if (orderQueue == NULL) {
        orderQueue = newOrder;
    } else {
        Order *last = orderQueue;
        while (last->next != NULL) last = last->next;
        last->next = newOrder;
    }


    currentCustomer.cart = NULL;

    printf("\n===== Order Confirmed! =====");
    printf("\nOrder ID: %d", newOrder->orderId);
    printf("\nDelivery to: %s", newOrder->customerAddress);
    printf("\nPayment: %s", newOrder->paymentMethod);
    printf("\nStatus: %s\n", newOrder->status);
}

void cancelOrder() {
    int orderId;
    printf("\nEnter Order ID to cancel: ");
    scanf("%d", &orderId);

    Order *prev = NULL;
    Order *current = orderQueue;

    while (current != NULL) {
        if (current->orderId == orderId) {

            for (int i = 0; i < current->productCount; i++) {
                for (int j = 0; j < productCount; j++) {
                    if (availableProducts[j].id == current->products[i].id) {
                        availableProducts[j].quantity += current->products[i].quantity;break;
                    }
                }
            }


            if (prev == NULL) {
                orderQueue = current->next;
            } else {
                prev->next = current->next;
            }

            free(current->products);
            free(current);
            printf("Order %d cancelled successfully!\n", orderId);
            return;
        }
        prev = current;
        current = current->next;
    }
    printf("Order not found or already processed!\n");
}


void assignDelivery() {
    if (orderQueue == NULL) {
        printf("\nNo orders in queue!\n");
        return;
    }

    if (staffStack == NULL) {
        printf("\nNo delivery staff available!\n");
        return;
    }


    Order *order = orderQueue;
    orderQueue = orderQueue->next;


    DeliveryStaff *staff = staffStack;
    staffStack = staffStack->next;


    Delivery *newDelivery = (Delivery *)malloc(sizeof(Delivery));
    newDelivery->deliveryId = 5000 + rand() % 5000;
    newDelivery->order = order;
    strcpy(newDelivery->currentLocation, "Daffodil Smart City");
    newDelivery->prev = NULL;
    newDelivery->next = deliveryList;
    if (deliveryList != NULL) deliveryList->prev = newDelivery;
    deliveryList = newDelivery;


    strcpy(order->status, "Processing");

    printf("\n===== Delivery Assigned =====");
    printf("\nOrder ID: %d", order->orderId);
    printf("\nAssigned to: %s (Staff ID: %d)", staff->name, staff->staffId);
    printf("\nO.T.P : %d\n", newDelivery->deliveryId);
}


void trackDelivery() {
    int orderId;
    printf("\nEnter Order ID to track: ");
    scanf("%d", &orderId);

    Delivery *delivery = deliveryList;
    while (delivery != NULL) {
        if (delivery->order->orderId == orderId) {
            printf("\n===== Order Tracking =====");
            printf("\nOrder ID: %d", delivery->order->orderId);
            printf("\nCustomer: %s", delivery->order->customerName);
            printf("\nStatus: %s", delivery->order->status);
            printf("\nCurrent Location: %s", delivery->currentLocation);
            printf("\nPayment Method: %s\n", delivery->order->paymentMethod);
            return;
        }
        delivery = delivery->next;
    }

    Order *order = orderQueue;
    while (order != NULL) {
        if (order->orderId == orderId) {
            printf("\nOrder ID: %d is still in queue", orderId);
            printf("\nStatus: %s\n", order->status);
            return;
        }
        order = order->next;
    }

    printf("\nOrder not found!\n");
}


void generateReport() {
    printf("\n===== System Report =====");


    printf("\n\n=== Products (%d) ===", productCount);
    for (int i = 0; i < productCount; i++) {
        printf("\nID: %d | %s | Category: %s", availableProducts[i].id,
               availableProducts[i].name, availableProducts[i].category);
        printf("\nPrice: $%.2f | Stock: %d", availableProducts[i].price,
               availableProducts[i].quantity);
        printf("\n--------------------------------");
    }


    printf("\n\n=== Orders ===");
    Order *order = orderQueue;
    int queueCount = 0, processingCount = 0, deliveredCount = 0;

    while (order != NULL) {
        printf("\nOrder ID: %d | Customer: %s", order->orderId, order->customerName);
        printf("\nStatus: %s | Items: %d", order->status, order->productCount);
        printf("\n--------------------------------");

        if (strcmp(order->status, "Queue") == 0) queueCount++;
        else if (strcmp(order->status, "Processing") == 0)
            processingCount++;
        else if (strcmp(order->status, "Delivered") == 0)
            deliveredCount++;

        order = order->next;
    }
    printf("\nQueue: %d | Processing: %d | Delivered: %d",
           queueCount, processingCount, deliveredCount);


    printf("\n\n=== Delivery Staff ===");
    DeliveryStaff *staff = staffStack;
    while (staff != NULL) {
        printf("\nID: %d | Name: %s", staff->staffId, staff->name);
        staff = staff->next;
    }
    printf("\n");
}
