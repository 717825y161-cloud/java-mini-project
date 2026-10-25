# java-mini-project
Online Auction &amp;amp; Bidding
```java
import java.util.*;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

class Seller {
    int id;
    String name;

    Seller(int id, String name) {
        this.id = id;
        this.name = name;
    }
}

class AuctionItem {
    int id;
    String name;
    String description;
    Seller seller;
    boolean transferred;

    AuctionItem(int id, String name, String description, Seller seller) {
        this.id = id;
        this.name = name;
        this.description = description;
        this.seller = seller;
        this.transferred = false;
    }
}

class Bidder {
    int id;
    String name;

    Bidder(int id, String name) {
        this.id = id;
        this.name = name;
    }
}

class Bid {
    int id;
    Auction auction;
    Bidder bidder;
    double amount;
    LocalDateTime time;

    Bid(int id, Auction auction, Bidder bidder, double amount) {
        this.id = id;
        this.auction = auction;
        this.bidder = bidder;
        this.amount = amount;
        this.time = LocalDateTime.now();
    }
}

class Winner {
    Bidder bidder;
    double finalPrice;
    boolean confirmed;

    Winner(Bidder bidder, double finalPrice) {
        this.bidder = bidder;
        this.finalPrice = finalPrice;
        this.confirmed = false;
    }
}

class Settlement {
    int id;
    Auction auction;
    Winner winner;
    boolean paid;
    LocalDateTime paymentTime;

    Settlement(int id, Auction auction, Winner winner) {
        this.id = id;
        this.auction = auction;
        this.winner = winner;
        this.paid = false;
    }

    void processPayment() {
        if (!winner.confirmed) {
            System.out.println("Winner confirmation is required before payment.");
            return;
        }

        if (paid) {
            System.out.println("Payment has already been completed.");
            return;
        }

        paid = true;
        paymentTime = LocalDateTime.now();
        auction.item.transferred = true;

        System.out.println("\n========== SETTLEMENT RECEIPT ==========");
        System.out.println("Settlement ID : " + id);
        System.out.println("Auction ID    : " + auction.id);
        System.out.println("Item          : " + auction.item.name);
        System.out.println("Seller        : " + auction.item.seller.name);
        System.out.println("Winner        : " + winner.bidder.name);
        System.out.printf("Final Price   : %.2f%n", winner.finalPrice);
        System.out.println("Payment       : Successful");
        System.out.println("Item Transfer : Completed");
        System.out.println("Payment Time  : " +
                paymentTime.format(DateTimeFormatter.ofPattern("dd-MM-yyyy HH:mm:ss")));
        System.out.println("========================================");
    }
}

class Auction {
    int id;
    AuctionItem item;
    double startingPrice;
    LocalDateTime startTime;
    LocalDateTime endTime;
    List<Bid> bids;
    boolean closed;
    Winner winner;
    Settlement settlement;

    Auction(int id, AuctionItem item, double startingPrice,
            LocalDateTime startTime, LocalDateTime endTime) {

        this.id = id;
        this.item = item;
        this.startingPrice = startingPrice;
        this.startTime = startTime;
        this.endTime = endTime;
        this.bids = new ArrayList<>();
        this.closed = false;
    }

    double getHighestBid() {
        double highest = startingPrice;

        for (Bid bid : bids) {
            if (bid.amount > highest) {
                highest = bid.amount;
            }
        }

        return highest;
    }

    boolean isWithinWindow() {
        LocalDateTime now = LocalDateTime.now();

        return !now.isBefore(startTime) && now.isBefore(endTime);
    }

    void placeBid(Bid bid) {
        if (closed) {
            System.out.println("Auction is already closed.");
            return;
        }

        if (!isWithinWindow()) {
            System.out.println("Bidding is not allowed outside the auction window.");
            return;
        }

        if (bid.amount <= getHighestBid()) {
            System.out.printf("Bid rejected! Bid must exceed %.2f%n",
                    getHighestBid());
            return;
        }

        bids.add(bid);

        System.out.println("Bid placed successfully!");
        System.out.println("Bidder       : " + bid.bidder.name);
        System.out.printf("Bid Amount   : %.2f%n", bid.amount);
        System.out.printf("Highest Bid  : %.2f%n", getHighestBid());
    }

    void closeAuction() {
        if (closed) {
            System.out.println("Auction is already closed.");
            return;
        }

        if (LocalDateTime.now().isBefore(endTime)) {
            System.out.println("Auction cannot be closed before its end time.");
            return;
        }

        closed = true;

        System.out.println("\n========== AUCTION CLOSED ==========");
        System.out.println("Auction ID : " + id);
        System.out.println("Item       : " + item.name);

        if (bids.isEmpty()) {
            System.out.println("No valid bids received.");
            System.out.println("Winner     : None");
            System.out.println("Final Price: No sale");
        } else {
            Bid highestBid = bids.get(0);

            for (Bid bid : bids) {
                if (bid.amount > highestBid.amount) {
                    highestBid = bid;
                }
            }

            winner = new Winner(highestBid.bidder, highestBid.amount);

            System.out.println("Winner     : " + winner.bidder.name);
            System.out.printf("Final Price: %.2f%n", winner.finalPrice);
            System.out.println("Confirmation required before settlement.");
        }

        System.out.println("====================================");
    }
}

public class Main {
    static Scanner sc = new Scanner(System.in);

    static List<Seller> sellers = new ArrayList<>();
    static List<AuctionItem> items = new ArrayList<>();
    static List<Bidder> bidders = new ArrayList<>();
    static List<Auction> auctions = new ArrayList<>();
    static List<Settlement> settlements = new ArrayList<>();

    static int sellerId = 1;
    static int itemId = 1;
    static int bidderId = 1;
    static int auctionId = 1;
    static int bidId = 1;
    static int settlementId = 1;

    static DateTimeFormatter formatter =
            DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");

    static String readLine(String message) {
        System.out.print(message);
        return sc.nextLine().trim();
    }

    static int readInt(String message) {
        while (true) {
            try {
                return Integer.parseInt(readLine(message));
            } catch (NumberFormatException e) {
                System.out.println("Please enter a valid integer.");
            }
        }
    }

    static double readDouble(String message) {
        while (true) {
            try {
                double value = Double.parseDouble(readLine(message));

                if (!Double.isFinite(value)) {
                    System.out.println("Enter a finite number.");
                    continue;
                }

                return value;
            } catch (NumberFormatException e) {
                System.out.println("Please enter a valid number.");
            }
        }
    }

    static LocalDateTime readDateTime(String message) {
        while (true) {
            try {
                return LocalDateTime.parse(readLine(message), formatter);
            } catch (Exception e) {
                System.out.println("Invalid format. Use yyyy-MM-dd HH:mm");
            }
        }
    }

    static Seller findSeller(int id) {
        for (Seller seller : sellers) {
            if (seller.id == id) {
                return seller;
            }
        }
        return null;
    }

    static AuctionItem findItem(int id) {
        for (AuctionItem item : items) {
            if (item.id == id) {
                return item;
            }
        }
        return null;
    }

    static Bidder findBidder(int id) {
        for (Bidder bidder : bidders) {
            if (bidder.id == id) {
                return bidder;
            }
        }
        return null;
    }

    static Auction findAuction(int id) {
        for (Auction auction : auctions) {
            if (auction.id == id) {
                return auction;
            }
        }
        return null;
    }

    static void addSeller() {
        String name = readLine("Enter seller name: ");

        if (name.isEmpty()) {
            System.out.println("Seller name cannot be empty.");
            return;
        }

        Seller seller = new Seller(sellerId++, name);
        sellers.add(seller);

        System.out.println("Seller registered successfully. ID: " + seller.id);
    }

    static void addItem() {
        if (sellers.isEmpty()) {
            System.out.println("Add a seller first.");
            return;
        }

        System.out.println("\nAvailable Sellers:");
        for (Seller seller : sellers) {
            System.out.println(seller.id + ". " + seller.name);
        }

        int id = readInt("Enter seller ID: ");
        Seller seller = findSeller(id);

        if (seller == null) {
            System.out.println("Seller not found.");
            return;
        }

        String name = readLine("Enter item name: ");
        String description = readLine("Enter item description: ");

        if (name.isEmpty()) {
            System.out.println("Item name cannot be empty.");
            return;
        }

        AuctionItem item =
                new AuctionItem(itemId++, name, description, seller);

        items.add(item);

        System.out.println("Item added successfully. Item ID: " + item.id);
    }

    static void createAuction() {
        if (items.isEmpty()) {
            System.out.println("Add an item first.");
            return;
        }

        System.out.println("\nAvailable Items:");

        for (AuctionItem item : items) {
            System.out.println(item.id + ". " + item.name
                    + " | Seller: " + item.seller.name
                    + (item.transferred ? " | Transferred" : ""));
        }

        int id = readInt("Enter item ID: ");
        AuctionItem item = findItem(id);

        if (item == null || item.transferred) {
            System.out.println("Invalid item or item already transferred.");
            return;
        }

        for (Auction auction : auctions) {
            if (auction.item.id == item.id && !auction.closed) {
                System.out.println("This item already has an active auction.");
                return;
            }
        }

        double price = readDouble("Enter starting price: ");

        if (price < 0) {
            System.out.println("Starting price cannot be negative.");
            return;
        }

        LocalDateTime start = readDateTime(
                "Enter start time (yyyy-MM-dd HH:mm): ");

        LocalDateTime end = readDateTime(
                "Enter end time (yyyy-MM-dd HH:mm): ");

        if (!end.isAfter(start)) {
            System.out.println("End time must be after start time.");
            return;
        }

        if (!end.isAfter(LocalDateTime.now())) {
            System.out.println("Auction end time must be in the future.");
            return;
        }

        Auction auction =
                new Auction(auctionId++, item, price, start, end);

        auctions.add(auction);

        System.out.println("Auction created successfully. Auction ID: "
                + auction.id);
    }

    static void registerBidder() {
        String name = readLine("Enter bidder name: ");

        if (name.isEmpty()) {
            System.out.println("Bidder name cannot be empty.");
            return;
        }

        Bidder bidder = new Bidder(bidderId++, name);
        bidders.add(bidder);

        System.out.println("Bidder registered successfully. ID: " + bidder.id);
    }

    static void placeBid() {
        if (bidders.isEmpty() || auctions.isEmpty()) {
            System.out.println("Register a bidder and create an auction first.");
            return;
        }

        int aId = readInt("Enter auction ID: ");
        Auction auction = findAuction(aId);

        if (auction == null) {
            System.out.println("Auction not found.");
            return;
        }

        int bId = readInt("Enter bidder ID: ");
        Bidder bidder = findBidder(bId);

        if (bidder == null) {
            System.out.println("Bidder not found.");
            return;
        }

        double amount = readDouble("Enter bid amount: ");

        if (amount <= 0) {
            System.out.println("Bid amount must be positive.");
            return;
        }

        Bid bid = new Bid(bidId++, auction, bidder, amount);
        auction.placeBid(bid);
    }

    static void closeAuction() {
        int id = readInt("Enter auction ID to close: ");
        Auction auction = findAuction(id);

        if (auction == null) {
            System.out.println("Auction not found.");
            return;
        }

        auction.closeAuction();
    }

    static void settlePayment() {
        int id = readInt("Enter auction ID: ");
        Auction auction = findAuction(id);

        if (auction == null) {
            System.out.println("Auction not found.");
            return;
        }

        if (!auction.closed) {
            System.out.println("Close the auction before settlement.");
            return;
        }

        if (auction.winner == null) {
            System.out.println("No winner exists for this auction.");
            return;
        }

        if (auction.settlement != null && auction.settlement.paid) {
            System.out.println("This auction has already been settled.");
            return;
        }

        if (!auction.winner.confirmed) {
            System.out.println("Winner: " + auction.winner.bidder.name);
            System.out.printf("Final price: %.2f%n",
                    auction.winner.finalPrice);

            String confirm = readLine(
                    "Confirm winner before payment? (yes/no): ");

            if (!confirm.equalsIgnoreCase("yes")) {
                System.out.println("Winner confirmation cancelled.");
                return;
            }

            auction.winner.confirmed = true;
            System.out.println("Winner confirmed successfully.");
        }

        if (auction.settlement == null) {
            Settlement settlement =
                    new Settlement(settlementId++, auction, auction.winner);

            auction.settlement = settlement;
            settlements.add(settlement);
        }

        String pay = readLine("Confirm payment? (yes/no): ");

        if (!pay.equalsIgnoreCase("yes")) {
            System.out.println("Payment cancelled. Settlement is pending.");
            return;
        }

        auction.settlement.processPayment();
    }

    static void displayAuctions() {
        if (auctions.isEmpty()) {
            System.out.println("No auctions available.");
            return;
        }

        System.out.println("\n================ AUCTION LIST ================");

        for (Auction auction : auctions) {
            System.out.println("\nAuction ID : " + auction.id);
            System.out.println("Item       : " + auction.item.name);
            System.out.println("Seller     : " + auction.item.seller.name);
            System.out.printf("Starting Price: %.2f%n",
                    auction.startingPrice);
            System.out.printf("Highest Bid   : %.2f%n",
                    auction.getHighestBid());

            System.out.println("Start Time : " + auction.startTime.format(formatter));
            System.out.println("End Time   : " + auction.endTime.format(formatter));
            System.out.println("Status     : " +
                    (auction.closed ? "Closed" :
                            auction.isWithinWindow() ? "Open" :
                                    LocalDateTime.now().isBefore(auction.startTime)
                                            ? "Scheduled" : "Ended - Awaiting Closure"));

            if (auction.winner != null) {
                System.out.println("Winner     : " + auction.winner.bidder.name);
                System.out.printf("Final Price: %.2f%n",
                        auction.winner.finalPrice);
                System.out.println("Confirmed  : " +
                        auction.winner.confirmed);
            }

            System.out.println("Item Transfer: " +
                    (auction.item.transferred ? "Completed" : "Pending"));

            if (auction.settlement != null) {
                System.out.println("Payment Status: " +
                        (auction.settlement.paid ? "Paid" : "Pending"));
            }
        }

        System.out.println("================================================");
    }

    public static void main(String[] args) {
        while (true) {
            System.out.println("\n====== ONLINE AUCTION SYSTEM ======");
            System.out.println("1. Add Seller");
            System.out.println("2. Add Item");
            System.out.println("3. Create Auction");
            System.out.println("4. Register Bidder");
            System.out.println("5. Place Bid");
            System.out.println("6. Close Auction");
            System.out.println("7. Settle Payment");
            System.out.println("8. Display Auctions");
            System.out.println("9. Exit");

            int choice = readInt("Enter your choice: ");

            switch (choice) {
                case 1:
                    addSeller();
                    break;

                case 2:
                    addItem();
                    break;

                case 3:
                    createAuction();
                    break;

                case 4:
                    registerBidder();
                    break;

                case 5:
                    placeBid();
                    break;

                case 6:
                    closeAuction();
                    break;

                case 7:
                    settlePayment();
                    break;

                case 8:
                    displayAuctions();
                    break;

                case 9:
                    System.out.println("Thank you for using the Online Auction System!");
                    sc.close();
                    return;

                default:
                    System.out.println("Invalid choice. Please try again.");
            }
        }
    }
}
```
