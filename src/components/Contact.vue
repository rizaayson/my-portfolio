<script setup>

	import {Notyf} from 'notyf';
	// Import onMounted() from Vue to run code after the component is displayed on the page.
	// Import onBeforeUnmount() from Vue to run code before the component is removed from the page.
	import {ref, onMounted, onBeforeUnmount} from 'vue';
	const notyf = new Notyf;

	const name = ref("");
	const email = ref("");
	const message = ref("");
	const isLoading = ref(false);

	// Web3Forms Access Key used to authenticate form submissions.
	const WEB3FORMS_ACCESS_KEY = "2bbdb7e6-d16c-4b63-a4a0-28a505127bdb";

	// Email subject that will appear when a form submission is received.
	const subject = "New message from Portfolio Contact Form";

	// The submitForm() function handles the contact form submission.
    // It sends the form data to Web3Forms and displays a notification based on the result.
	const submitForm = async () => {

		// Ensure the user complets the reCAPTCHA challenge before submitting the form
		if(!recaptchaToken.value) {
			notyf.error('Please verify that you are not a robot');
			// Stop the form submission process
			return;
		}

		// Set the loading state to true while the form is being submitted.
        // While the email is being sent, disable the button and change its text to "Sending...". Revert it to "Submit" once complete.
		isLoading.value = true;

		try {

			// Send a POST request to the Web3Forms API endpoint.
            // fetch() API is a built-in JavaScript function used to send HTTP requests to a server.
			const response = await fetch("https://api.web3forms.com/submit", {
				method: "POST",
				headers: {
					"Content-Type": "application/json",
					// Indicates that the request accepts a JSON response.
					Accept: "application/json"
				},
				// Convert the form data into JSON string
				body: JSON.stringify({
					access_key: WEB3FORMS_ACCESS_KEY,
					subject: subject,
					name: name.value,
					email: email.value,
					message: message.value
				})
			})

			// Convert the API response into a JavaScript object.
			const result = await response.json();

			// Check if the form submission was successful.
			if (result.success) {
				console.log(result);
				isLoading.value = false;
				notyf.success("Message Sent!");
			}
		} catch (error) {
			console.log(error);
			isLoading.value = false;
			notyf.error("Failed to send message.");
		} finally {
			// Reset the reCAPTCHA widget after the submission process completes, whether the request succeeds or fails.
			resetRecaptcha();
		}
	}

	/*reCAPTCHA Integration*/

    const SITE_KEY = '6LdI2AUtAAAAALVbp_lybeBB5b9hCbujKh-ftVDA';  // Replace with your site key
    // The location where the reCAPTCHA checkbox will appear
    const recaptchaContainer = ref(null);
    // The ID of the reCAPTCHA widget after it is created
    const recaptchaWidgetId = ref(null);
    // Stores the token generated when the user completes reCAPTCHA
    const recaptchaToken = ref('');

    // Runs when the user successfully completes the reCAPTCH challenge
    function onRecaptchaSuccess(token) {
    	// Save the generated token
        recaptchaToken.value = token;
    }

    // Runs when the reCAPTCHA verification expires
    // For Google reCAPTCHA v2, the token typically expires after about 2 minutes (120 seconds) if the form has not been submitted.
    function onRecaptchaExpired() {
    	// Remove the expired token
        recaptchaToken.value = '';
    }

    // Creates and displays the reCAPTCHA widget
    function renderRecaptcha() {

    	// Check if the Google reCAPTCHA script is available
    	// window refers to the browser window object
        if (!window.grecaptcha) {
            console.error('reCAPTCHA not loaded');
            // Stop the function
            return;
        }

        // Create the reCAPTCH widget and save its ID
        // Google returns a unique widget ID after creating the reCAPTCHA widget
        // render() generates the reCAPTCHA checkbox inside recaptchaContainer.value
        recaptchaWidgetId.value = window.grecaptcha.render(recaptchaContainer.value, {
        	// Your Google reCAPTCHA Site Key
            sitekey: SITE_KEY,
            // Displays the standard reCAPTCHA checkbox, making it easier for users to see and interact with on most forms.
            size: 'normal',
            // Run this function when the user completes the reCAPTCHA challenge
            callback: onRecaptchaSuccess,
            // Run this function when the reCAPTCHA token expires
            'expired-callback': onRecaptchaExpired
        });
    }

    // Function that resets the reCAPTCHA widget
    function resetRecaptcha() {
    	// Check if the reCAPTCHA widget has already been created
        if (recaptchaWidgetId.value !== null) {
	       	// Reset the widget to its initial state
	        window.grecaptcha.reset(recaptchaWidgetId.value);
	        // Clear the stored verification token
	        recaptchaToken.value = '';
        }
    }

    // Run this code after the component is displayed on the page
    onMounted(() => {
    // This makes sure that the reCAPTCHA widget is rendered only after the library is available.
    // setInterval() is a JavaScript function that repeatedly executes a block of code at a specified time interval.
    const interval = setInterval(() => {
    	// Verify that the grecaptcha object and render() method are available
        if (window.grecaptcha && window.grecaptcha.render) {
        	// Create and display the reCAPTCHA widget
            renderRecaptcha();
            // Stop checking once the widget has been rendered
            clearInterval(interval);
        }
        // Check every 100 milliseconds if the Google reCAPTCHA library has loaded
    }, 100);

	    // Run cleanup code before the component is removed
		onBeforeUnmount(() => {
		// Stop the interval to prevent unnecessary checks
		clearInterval(interval);
		});
	});

	
</script>

<template>
	<!-- Start of Contact -->
	<h1 class="text-center my-4 pt-5" id="contact">Contact</h1>
	<div class="contact-section">
	    <div class="row align-items-center mt-4">
	        <div class="col-md-6 map-container">
	            <iframe id="gmap_canvas" src="https://maps.google.com/maps?q=centro%20escolar%20university%20manila&t=&z=13&ie=UTF8&iwloc=&output=embed" frameborder="0" scrolling="no" marginheight="0" marginwidth="0"></iframe>
	        </div>
	        <div class="col-md-6">
	        	<!-- Bind the submitForm() function to the form submit event. -->
                <!-- The .prevent modifier prevents the default form behavior and page refresh. -->
	            <form @submit.prevent="submitForm">
	                <div class="mb-3">
	                	<!-- Bind the input field to the name reactive variable using v-model. -->
                        <!-- Any value entered in this field is automatically stored in the name state. -->
	                    <input type="text" v-model="name" class="form-control contact-form-control" placeholder="First Name M.I. Last Name">
	                </div>
	                <div class="mb-3">
	                    <input type="email" v-model="email" class="form-control contact-form-control" placeholder="Email">
	                </div>
	                <div class="mb-3">
	                    <textarea class="form-control contact-form-control" v-model="message" rows="6" placeholder="Message"></textarea>
	                </div>
	                <div class="form-footer">
	                    <div class="social-icons">
	                        <a href="https://www.linkedin.com/in/charles-babbage-8291a6211/" id="linkedin"><i class="fab fa-linkedin"></i></a>
	                        <a href="https://gitlab.com/cbabbage0991" id="gitlab"><i class="fab fa-gitlab"></i></a>
	                        <a href="https://github.com/cbabbage0991" id="github"><i class="fab fa-github"></i></a>
	                    </div>
	                    <button type="submit" class="submit-btn pl-5 pr-5" :disabled="isLoading">{{isLoading ? "Sending..." : "Submit"}}</button>
	                </div>

	                <!-- The location where the reCAPTCHA checkbox will appear -->
	                <div class="d-flex justify-contend-end mt-2">
	                	<!-- Creates a reference to the div so the reCAPTCHA widget can be rendered into it. -->
	                	<div ref="recaptchaContainer"></div>
	                </div>
	            </form>
	            
	        </div>
	    </div>
	</div>
	<!-- End of Contact -->
</template>

<style scoped>
	
</style>
