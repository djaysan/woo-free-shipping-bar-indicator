WooCommerce Free Shipping Progress Bar
This snippet adds a Free Shipping Progress Bar to your WooCommerce Cart and Checkout pages.

It visually shows customers how close they are to reaching the free shipping threshold, helping increase average order value.

Features
✅ Progress bar automatically updates via AJAX when items are added to the cart
✅ Supports Cart and Checkout views
✅ Designed for Poland (PL) shipping zone — easily adaptable
✅ Compatible with the built-in WooCommerce Free Shipping method
✅ Simple, clean CSS included — ready to customize

How it works
Detects the minimum amount required for free shipping (from your PL zone "Free Shipping" method)

Calculates current cart total

Displays a progress bar with remaining amount, or a success message if free shipping is reached

Integrates seamlessly with WooCommerce

Installation
Add the provided PHP code to your theme's functions.php file
or use a custom plugin.

```
/**
 * Free shipping bar indicator (WooCommerce cart + checkout view)
 */

// Hide shipping calculator on Cart page only
add_filter( 'woocommerce_cart_needs_shipping', 'filter_cart_needs_shipping' );
function filter_cart_needs_shipping( $needs_shipping ) {
    if ( is_cart() ) {
        $needs_shipping = false;
    }
    return $needs_shipping;
}

// Make progress bar update with AJAX fragments
add_filter( 'woocommerce_add_to_cart_fragments', 'add_free_shipping_bar_to_fragments' );
function add_free_shipping_bar_to_fragments( $fragments ) {
    ob_start();
    display_free_shipping_progress_bar(); // Render progress bar HTML
    $fragments['.free-shipping-bar-container'] = ob_get_clean(); // Target container class
    return $fragments;
}

// Display on Cart page (above cart totals)
add_action( 'woocommerce_before_cart_collaterals', 'display_free_shipping_progress_bar' );

// Display on Checkout page (above checkout form)
add_action( 'woocommerce_before_checkout_form', 'display_free_shipping_progress_bar', 5 );

// Main function to display the progress bar
function display_free_shipping_progress_bar() {
    // Get all zones
    $zones = WC_Shipping_Zones::get_zones();

    // Find the zone for Poland (PL)
    $poland_zone = null;
    foreach ( $zones as $zone_data ) {
        foreach ( $zone_data['zone_locations'] as $location ) {
            if ( 'PL' === $location->code && 'country' === $location->type ) {
                $poland_zone = $zone_data;
                break 2; // Exit both loops when found
            }
        }
    }

    if ( ! $poland_zone ) {
        return; // Exit if no zone for PL
    }

    // Find Free Shipping method in the zone
    $free_shipping_method = null;
    foreach ( $poland_zone['shipping_methods'] as $method ) {
        if ( 'free_shipping' === $method->id ) {
            $free_shipping_method = $method;
            break;
        }
    }

    if ( ! $free_shipping_method ) {
        return; // Exit if Free Shipping is not available
    }

    // Get minimum amount for Free Shipping
    $min_amount = isset( $free_shipping_method->instance_settings['min_amount'] ) ? $free_shipping_method->instance_settings['min_amount'] : 0;

    if ( $min_amount <= 0 ) {
        return; // Exit if no minimum amount is set
    }

    // Get cart total
    $cart_total = WC()->cart->get_displayed_subtotal();
    $ignore_discounts = $free_shipping_method->settings['ignore_discounts'] ?? 'no';

    if ( 'no' === $ignore_discounts ) {
        $cart_total -= WC()->cart->get_discount_total();
        if ( WC()->cart->display_prices_including_tax() ) {
            $cart_total -= WC()->cart->get_discount_tax();
        }
    }

    // Calculate progress + message
    if ( $cart_total >= $min_amount ) {
        $progress = 100;
        $message  = __( 'Gratulacje! Masz darmową dostawę!', 'woocommerce' );
    } else {
        $remaining = $min_amount - $cart_total;
        $progress  = ( $cart_total / $min_amount ) * 100;
        $message   = sprintf(
            /* translators: %s: amount remaining for free shipping */
            __( 'Do darmowej dostawy brakuje ci %s', 'woocommerce' ),
            '<strong>' . number_format( $remaining, 2, ',', '' ) . ' zł</strong>'
        );
    }

    // Display progress bar (no SVG icon)
    echo '<div class="free-shipping-bar-container">';
    echo '<div class="free-shipping-bar">';
    echo '<p>' . wp_kses_post( $message ) . '</p>';
    echo '<div class="progress-bar-container">';
    echo '<div class="progress-bar" style="width: ' . esc_attr( $progress ) . '%;"></div>';
    echo '</div>';
    echo '</div>';
    echo '<div class="clearfix"></div>';
    echo '</div>';
}

```

Add the provided CSS to your theme’s stylesheet or via Customizer > Additional CSS.

```
/* Free shipping bar */

.free-shipping-bar {
    margin: 30px 0 24px 0;
    padding: 35px;
    background-color: #fff;
    border: 0px solid #dcdcdc;
    border-radius: 0px;
	box-shadow: 0px 30px 60px 0px rgba(0, 0, 0, 0.1);
    color: #848484;
}

@media (min-width: 768px) {
    .free-shipping-bar {
        width: 100%;
        float: right;
    }
}

.free-shipping-bar p {
    margin: 0 0 10px 0;
}

.free-shipping-bar .progress-bar-container {
    margin-top: 10px;
    background-color: #f2f2f2;
    border: 0px solid #dfdfdf;
    border-radius: 20px;
    height: 20px;
    overflow: hidden;
}

.free-shipping-bar .progress-bar {
    background-color: #f80;
    height: 100%;
}

.free-shipping-bar span.svgIcon {
    float: left;
    margin: 0 15px 0 0;
    top: -1px;
    position: relative;
}

.free-shipping-bar strong {
    color: #323232;
}
```

Example Messages
Before reaching free shipping:
Do darmowej dostawy brakuje ci 25,00 zł

After reaching free shipping:
Gratulacje! Masz darmową dostawę!

Notes
This version is configured for the Poland (PL) shipping zone.

You can adapt it to other zones by adjusting the country code.

The progress bar targets .free-shipping-bar-container for AJAX updates.

No SVG icons are included, but the CSS supports adding one if needed.


Compatibility
WooCommerce 5.x, 6.x, 7.x, 8.x+

Tested with standard WooCommerce Cart & Checkout templates

Works with most themes (may require minor CSS adjustments)
