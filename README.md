# export-featured-images-csv
A one time run script to export and download all the posts with their respective featured images in a csv file. 

<?php
require_once('wp-load.php');

global $wpdb;

// Open a file in write mode ('w')
$fp = fopen('exported_posts_images.csv', 'w');

// Add CSV headers
fputcsv($fp, ['Post ID', 'Post Title', 'Featured Image URL']);

$posts = $wpdb->get_results("
    SELECT p.ID, p.post_title, pm.meta_value as '_thumbnail_id'
    FROM {$wpdb->prefix}posts p
    LEFT JOIN {$wpdb->prefix}postmeta pm ON p.ID = pm.post_id AND pm.meta_key = '_thumbnail_id'
    WHERE p.post_type = 'post' AND p.post_status = 'publish'
");

foreach ($posts as $post) {
    if (!empty($post->_thumbnail_id)) {
        $image_url = wp_get_attachment_url($post->_thumbnail_id);
        if ($image_url) {
            fputcsv($fp, [$post->ID, $post->post_title, $image_url]);
        }
    }
}

// Close the file
fclose($fp);

echo "Export complete. CSV generated at 'exported_posts_images.csv'.";
?>
