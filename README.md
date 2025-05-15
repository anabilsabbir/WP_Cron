# WP_Cron Project

Step: 1 

register_activation_hook( __FILE__, 'bes_schedule_cron_event' );

Description:
  * This line tells WordPress to run a function when the plugin is activated.
  * The function being called is: bes_schedule_cron_event.

Step 2

function bes_schedule_cron_event() {
    if ( ! wp_next_scheduled( 'bes_send_batch_email_to_users' ) ) {
        wp_schedule_event( time(), 'every_minutes', 'bes_send_batch_email_to_users' );
    }
}

Description:

  * This function checks: "Is the cron job already scheduled?"
  * If not, it schedules a new cron job that:
    * Starts now
    * Repeats every minute (custom interval)
    * Triggers the action: 'bes_send_batch_email_to_users'

Step: 3 

register_deactivation_hook( __FILE__, 'bes_remove_cron_event' );

function bes_remove_cron_event() {
    wp_clear_scheduled_hook( 'bes_send_batch_email_to_users' );
    delete_option( 'bes_email_batch_offset' );
}

Description:

  * When the plugin is deactivated, this function runs.
  * It:
    * Removes the scheduled cron event.
    * Deletes the saved offset (used to keep track of which users have already been emailed).

Step: 4

add_filter( 'cron_schedules', 'bes_custom_cron_intervals' );
function bes_custom_cron_intervals( $schedules ) {
    $schedules['every_minutes'] = array(
        'interval' => 60, 
        'display'  => esc_html__( 'Every Minutes' ),
    );
    return $schedules;
}

Description:

  * WordPress by default doesn't support every-minute cron jobs.
  * So this code adds a new custom interval called 'every_minutes' that runs every 60 seconds.


Step: 5

add_action( 'bes_send_batch_email_to_users', 'bes_send_email_batch_to_users' );

Description: 

  * This tells WordPress:
    * When 'bes_send_batch_email_to_users' runs (by the cron), call the function: bes_send_email_batch_to_users.

Step: 6

add_action( 'bes_send_batch_email_to_users', 'bes_send_email_batch_to_users' );

function bes_send_email_batch_to_users() {
    $batch_size = 10;
    $offset = get_option( 'bes_email_batch_offset', 0 );

    $users = get_users( array(
        'number'  => $batch_size,
        'offset'  => $offset,
        'orderby' => 'ID',
        'order'   => 'ASC',
        'fields'  => array( 'ID', 'user_email', 'display_name' ),
    ) );

    if ( empty( $users ) ) {
        
        wp_clear_scheduled_hook( 'bes_send_batch_email_to_users');
        delete_option( 'bes_email_batch_offset' );
        return;
    }

    foreach ( $users as $user ) {
        $email   = $user->user_email;
        $name    = $user->display_name;
        $subject = 'Scheduled Email';
        $message = "Hi $name,\n\nThis is a scheduled batch email.";
        $headers = array( 'Content-Type: text/html; charset=UTF-8' );

        wp_mail( $email, $subject, nl2br( $message ), $headers );
    }

    update_option( 'bes_email_batch_offset', $offset + $batch_size );
}


























