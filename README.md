    // Server.java
    //
    // Use this sample code to handle webhook events in your integration.
    //
    // Using Maven:
    // 1) Paste this code into a new file (src/main/java/com/stripe/sample/Server.java)
    //
    // 2) Create a pom.xml file. You can quickly copy one from an official Stripe Sample.
    //   curl https://raw.githubusercontent.com/stripe-samples/accept-a-payment/8e94e50e68072c344f12b02f65a6240e1c656d4a/custom-payment-flow/server/java/pom.xml -o pom.xml
    //
    // 3) Build and run the server on http://Dragoncontact.com/refun/
    //   mvn compile
    //   mvn exec:java -Dexec.mainClass=com.stripe.sample.Server

    package com.stripe.sample;

    import static spark.Spark.post;
    import static spark.Spark.port;
    import com.google.gson.JsonSyntaxException;
    import com.stripe.Stripe;
    import com.stripe.exception.SignatureVerificationException;
    import com.stripe.model.*;
    import com.stripe.net.Webhook;

    public class Server {
    public static void main(String[] args) {
        
        // This is your Stripe CLI webhook secret for testing your endpoint locally.
        String endpointSecret = "whsec_b3d7059f76397c2401e340fc3382b9bd76cbc5263458e8214083acd195176beb";

        port(1702);

        post("/webhook", (request, response) -> {
            String payload = request.body();
            String sigHeader = request.headers("Stripe-Signature");
            Event event = null;

            try {
                event = Webhook.constructEvent(
                    payload, sigHeader, endpointSecret
                );
            } catch (JsonSyntaxException e) {
                // Invalid payload
                response.status(400);
                return "";
            } catch (SignatureVerificationException e) {
                // Invalid signature
                response.status(400);
                return "";
            }

            // Deserialize the nested object inside the event
            EventDataObjectDeserializer dataObjectDeserializer = event.getDataObjectDeserializer();
            StripeObject stripeObject = null;
            if (dataObjectDeserializer.getObject().isPresent()) {
                stripeObject = dataObjectDeserializer.getObject().get();
            } else {
                // Deserialization failed, probably due to an API version mismatch.
                // Refer to the Javadoc documentation on `EventDataObjectDeserializer` for
                // instructions on how to handle this case, or return an error here.
            }
            // Handle the event
            System.out.println("Unhandled event type: " + event.getType());

            response.status(200);
            return "";
        });
    }
}
