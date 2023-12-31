<script>
	//@ts-nocheck
	import PaylnSvg from '$lib/PaylnSVG.svelte';
	import Input from '$lib/Input.svelte';
	import { pageLoading } from '$lib/store';
	import { goto } from '$app/navigation';
	import { signupResult } from '$lib/store';

	const emailRegex = /^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$/;

	let formData = {
		first_name: '',
		last_name: '',
		email: '',
		password: '',
		confirmPassword: ''
	};

	let passwordMatch = '';
	let emailCheck = '';

	async function formSubmit() {
		switch (true) {
			case emailRegex.test(formData.email) === false:
				console.log('Invalid email');
				emailCheck = 'Invalid email';
				break;
			case formData.password !== formData.confirmPassword || formData.password === '':
				console.log('Password and confirm password do not match');
				passwordMatch = 'Password and confirm password do not match';
				break;
			case formData.password.length < 8:
				console.log('Password is too short');
				passwordMatch = 'Password is too short';
				break;
			default:
				pageLoading.set(true);
				const response = await fetch('https://payln-staging.onrender.com/auth/signup', {
					method: 'POST',
					headers: {
						'Content-Type': 'application/json'
					},
					body: JSON.stringify(formData)
				});
				const result = await response.json();
				console.log('result = ', result);
				signupResult.set(result.data);
				if (result.errors) {
					pageLoading.set(false);
					signupResult.set(result.errors);
				} else if (result !== null && result.status !== 'error') {
					goto('verification');
					setTimeout(() => {
						pageLoading.set(false);
					}, 1000);
				} else if (result.status === 'error') {
					goto('login');
					setTimeout(() => {
						pageLoading.set(false);
					}, 1000);
				}

				break;
		}
	}

	let passType = 'password';

	const passVisibility = () => {
		passType = passType === 'password' ? 'text' : 'password';
	};

	function keyDown(event) {
		if (event.key === 'Enter') {
			passVisibility();
		}
	}

	const formCss =
		'block py-2.5 px-0 w-full text-sm text-gray-900 bg-transparent border-0 border-b-2 border-gray-300 appearance-none focus:outline-none focus:ring-0 focus:border-blue-600 peer';

	const passwordField = [
		{
			label: 'Password',
			type: 'password',
			name: 'password',
			id: 'password',
			style: formCss
		},
		{
			label: 'Confirm password',
			type: 'password',
			name: 'confirmPassword',
			id: 'floating_confirm_password',
			style: formCss
		}
	];

	const nameField = [
		{
			name: 'first_name',
			id: 'floating_first_name',
			label: 'First name',
			type: 'text',
			style: formCss
		},
		{
			name: 'last_name',
			id: 'floating_last_name',
			label: 'Last name',
			type: 'text',
			style: formCss
		},
		{
			name: 'email',
			id: 'floating_email',
			label: 'Email address',
			type: 'email',
			style: formCss
		}
	];
</script>

<section class="p-2 sm:p-6 md:p-10">
	<div class="flex flex-col items-center sm:items-stretch sm:w-full sm:flex-row sm:justify-center">
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
		<!-- Main Form -->
		<div
			class="w-full p-4 max-w-sm bg-white border border-gray-200 border-r-transparent rounded-b-lg sm:rounded-l-lg shadow sm:p-6 md:p-8"
		>
			<h3 class="md:text-3xl text-xl capitalize font-medium text-[#223d5b] pb-4">
				Sign up for PayLN
			</h3>
			<form on:submit|preventDefault={formSubmit}>
				<!-- Names and Email -->
				{#each nameField as field}
					<div class="relative z-0 w-full mb-3 group">
						<Input
							type={field.type}
							name={field.name}
							id={field.id}
							placeholder=""
							required
							bind:value={formData[field.name]}
							style={field.style}
						/>
						<label
							for={field.id}
							class="peer-focus:font-medium absolute text-sm text-gray-500 duration-300 transform -translate-y-6 scale-75 top-3 -z-10 origin-[0] peer-focus:left-0 peer-focus:text-blue-600 peer-placeholder-shown:scale-100 peer-placeholder-shown:translate-y-0 peer-focus:scale-75 peer-focus:-translate-y-6"
						>
							{field.label}
						</label>
					</div>
				{/each}
				<!-- End of Names and Email -->
				<p class="mb-3 text-xs text-red-600">{emailCheck}</p>
				<!-- Password -->
				{#each passwordField as field}
					<div class="relative z-0 w-full mb-4 group">
						<Input
							type={passType}
							name={field.name}
							id={field.id}
							placeholder=""
							required
							bind:value={formData[field.name]}
							style={field.style}
						/>
						<!-- Password Visibility Icon -->
						<span class="absolute top-1 mt-3 right-0 flex items-center px-2 cursor-pointer">
							<i
								on:click={passVisibility}
								on:keydown={keyDown}
								class:fa-eye-slash={passType === 'text'}
								class:fa-eye={passType !== 'text'}
								class="fas"
								aria-label={passType === 'text' ? 'Hide password' : 'Show password'}
								role="button"
								tabindex="0"
							/>
						</span>
						<!-- End Password Visibility Icon -->
						<label
							for={field.id}
							class="peer-focus:font-medium absolute text-sm text-gray-500 duration-300 transform -translate-y-6 scale-75 top-3 -z-10 origin-[0] peer-focus:left-0 peer-focus:text-blue-600 peer-placeholder-shown:scale-100 peer-placeholder-shown:translate-y-0 peer-focus:scale-75 peer-focus:-translate-y-6"
						>
							{field.label}
						</label>
						<p id="standard_error_help" class="mt-2 text-xs text-red-600">
							{passwordMatch}
						</p>
					</div>
				{/each}
				<!-- End of Password -->
				<!-- Buttons -->
				<div class="flex flex-row mt-12 justify-between">
					<button
						type="submit"
						on:click={formSubmit}
						class="text-white bg-[#223d5b] hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm w-auto px-5 py-2.5 text-center"
						>Submit</button
					>
					<button
						type="button"
						class="text-white bg-[#223d5b] hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm w-auto px-5 py-2.5 text-center"
					>
						<a data-sveltekit-preload-data="hover" href="login">Login</a>
					</button>
				</div>
			</form>
		</div>
		<!-- End of Main Form -->
		<!-- Side Logo -->
		<div
			class="hidden sm:block w-full p-4 max-w-sm bg-[#223d5b] border border-gray-200 border-l-transparent rounded-r-lg shadow sm:p-6 md:p-8"
		>
			<div
				class="flex align-middle justify-center items-center content-center mx-auto text-center w-[15rem] h-full"
			>
				<a href="/">
					<PaylnSvg />
				</a>
			</div>
		</div>
		<!-- End of Side Logo -->
	</div>
</section>
