<script>
	// @ts-nocheck
	import Input from '$lib/Input.svelte';
	import PaylnSvg from '$lib/PaylnSVG.svelte';
	import { signupResult, pageLoading } from '$lib/store';

	const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

	const form_css =
		'bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5';

	const password_field = {
		label: 'Password',
		type: 'password',
		name: 'password',
		id: 'password',
		placeholder: '••••••••',
		style: form_css
	};

	let email = '';
	let password = '';
	let password_Match = '';
	let email_check = '';
	let data;

	// function to change the visibility of the password
	let pass_Type = 'password';
	const pass_Visibility = () => {
		pass_Type = pass_Type === 'password' ? 'text' : 'password';
	};

	// keydown for func to change the visibility of the password, 11ty needed.
	function KeyDown(event) {
		if (event.key === 'Enter') {
			pass_Visibility();
		}
	}

	signupResult.subscribe((value) => {
		data = value;
	});

	async function formSubmit() {
		if (emailRegex.test(email) === false) {
			console.log('Invalid email');
			email_check = 'Invalid email';
			return;
		}

		if (password.length < 8) {
			console.log('Password is too short');
			password_Match = 'Password is too short';
			return;
		}

		pageLoading.set(true);

		try {
			const response = await fetch('https://payln-staging.onrender.com/auth/login', {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json'
				},
				body: JSON.stringify({
					email,
					password
				})
			});

			const result = await response.json();

			signupResult.set(result);
			localStorage.setItem('signupResult', JSON.stringify(result.data));
			console.log(result);

			if (result !== null && result.status === 'info') {
				window.location.href = 'verification';
			}

			if (result !== null && result.status === 'success') {
				alert('Successful!');
				data.message = "You've Successfully Logged in";
			}

			if (result !== null && result.status === 'error') {
				alert('Unsuccessful!');
				data.message = 'You were Unsuccessful in Logging in';
			}
		} catch (error) {
			console.error(error);
		} finally {
			pageLoading.set(false);
		}
	}

	const passwordInputProps = {
		label: password_field.label,
		type: pass_Type,
		name: password_field.name,
		id: password_field.id,
		placeholder: password_field.placeholder,
		style: password_field.style
	};
</script>

<div class="pt-10">
	<div class="p-2 mx-auto sm:p-6 md:p-10">
		<section
			class="flex flex-col items-center sm:items-stretch sm:w-full sm:flex-row sm:justify-center"
		>
			<!-- Top Logo -->
			<div
				class="w-full h-[10vh] sm:hidden p-2 max-h-sm max-w-sm bg-[#223d5b] border border-gray-200 rounded-t-lg shadow flex justify-center"
			>
				<div class="w-[6rem] mt-[1rem]">
					<a href="/">
						<PaylnSvg />
					</a>
				</div>
			</div>
			<!-- End of Top Logo  -->
			<!-- Email -->
			<div
				class="w-full p-4 max-w-sm bg-white border border-gray-200 border-r-transparent rounded-b-lg sm:rounded-l-lg shadow sm:p-6 md:p-8"
			>
				<form class="space-y-12" action="#">
					<h5 class="md:text-3xl text-xl font-medium text-[#223d5b] text-center">Login to PayLN</h5>
					<div>
						<label for="email" class="block mb-2 text-sm font-medium text-gray-900"
							>Your email</label
						>
						<input
							type="email"
							name="email"
							id="email"
							bind:value={email}
							class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5"
							placeholder="name@company.com"
							required
						/>
						<p class="mt-2 ml-2 text-xs text-red-600">
							{email_check}
						</p>
					</div>
					<!-- End of Email -->
					<!-- Password -->
					<div class="relative">
						<label for="password" class="block mb-2 text-sm font-medium text-gray-900"
							>Your password</label
						>
						<!-- Password Visibility Icon -->
						<span class="absolute bottom-3 right-0 flex items-center px-2 cursor-pointer">
							<i
								on:click={pass_Visibility}
								on:keydown={KeyDown}
								class:fa-eye-slash={pass_Type == 'text'}
								class:fa-eye={pass_Type !== 'text'}
								class="fas"
								aria-label={pass_Type == 'text' ? 'Hide password' : 'Show password'}
								role="button"
								tabindex="0"
							/>
						</span>

						<!-- End Password Visibility Icon -->
						<Input
							type={passwordInputProps.type}
							name={passwordInputProps.name}
							id={passwordInputProps.id}
							placeholder={passwordInputProps.placeholder}
							required
							bind:value={password}
							style={passwordInputProps.style}
						/>
						<p class="mt-2 ml-2 text-xs text-red-600">
							{password_Match}
						</p>
					</div>
					<!-- End of Password -->
					<!-- Buttons -->
					<div class="flex mx-auto items-start">
						<a href="#" class="mx-auto text-sm text-[#223d5b] hover:underline">Lost Password?</a>
					</div>
					<button
						type="submit"
						on:click={formSubmit}
						class="w-full text-white bg-[#223d5b] hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 text-center"
						>Login to your account</button
					>
					<div class="text-sm mx-auto w-fit font-medium text-gray-900">
						Not registered? <a href="signup" class="text-[#223d5b] hover:underline"
							>Create account</a
						>
					</div>
				</form>
				{#if data}
					<h2 class="mt-4 text-md text-red-600">{data.message}</h2>
				{/if}
			</div>

			<!-- Side Logo -->
			<div
				class="hidden sm:block w-full p-4 max-w-md bg-[#223d5b] border border-gray-200 border-l-transparent rounded-r-lg shadow sm:p-6 md:p-8"
			>
				<div class="pt-[20vh]">
					<a href="/">
						<PaylnSvg />
					</a>
				</div>
			</div>
			<!-- End of Side Logo -->
		</section>
	</div>
</div>
