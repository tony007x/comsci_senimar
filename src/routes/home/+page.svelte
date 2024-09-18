<script lang="ts">
    import { onMount } from "svelte";
    import wretch from "wretch";
    import { ScrollArea } from "$lib/components/ui/scroll-area/index.js";
    import { Trash2 } from "lucide-svelte";
    import { Button, buttonVariants } from "$lib/components/ui/button/index.js";
    import * as Dialog from "$lib/components/ui/dialog/index.js";
    import { Input } from "$lib/components/ui/input/index.js";
    import { Label } from "$lib/components/ui/label/index.js";
    import type { Value } from "$lib/components/ui/select";
    import { toast } from "svelte-sonner";

    interface User {
        id: number;
        fname: string;
        lname: string;
    }
    let isOpen = false;

    function closeDialog() {
        isOpen = false;
    }

    function openDialog() {
        isOpen = true;
    }

    // Fetch Data
    let userData: User[] = [];
    let isLoading: boolean = true; // Add loading state

    onMount(async () => {
        try {
            userData = await wretch(
                "https://seminarbackend.vercel.app/user/show",
            )
                .get()
                .json<User[]>();
            console.log(userData);
        } catch (error) {
            console.error("Error fetching user data:", error);
        } finally {
            isLoading = false; // Set loading to false after data is fetched
        }
    });

    // Add User
    let fname: string;
    let lname: string;
    let newfname: string;
    let newlname: string;
    const click = async () => {
        const res = wretch("https://seminarbackend.vercel.app/user/reg")
            .post({
                fname: fname,
                lname: lname,
            })
            .res(() => {
                toast.success("Add new user successfully!");
                sleep(3000);
                window.location.reload();
            });
    };

    const edit = async () => {
        const id = document.getElementById("id_user") as HTMLInputElement;
        wretch("https://seminarbackend.vercel.app/user/edit")
            .put({
                id: id.value,
                newfname: newfname,
                newlname: newlname,
            })
            .res(async () => {
                toast.success("Update Successfully!");
                await sleep(3000);
                window.location.reload();
            });
    };

    const deleteuser = async () => {
        const id = (document.getElementById("id_user") as HTMLInputElement)
            .value;
        await wretch(`https://seminarbackend.vercel.app/user/delete/${id}`)
            .delete()
            .res(async () => {
                toast.success("Deleted");
                await sleep(3000);
                window.location.reload();
            });
    };

    function sleep(ms: number): Promise<void> {
        return new Promise((resolve) => setTimeout(resolve, ms));
    }
</script>

<div class="flex  w-full h-screen justify-between ">
    <div class="flex w-full max-lg:flex-col">
    <!-- LEFT -->
    <div class="flex w-full justify-center p-5 items-center">
        <div
            class="flex flex-col w-[300px] shadow-lg shadow-slate-400 rounded-lg p-4 gap-3"
        >
            <label for="Fname">First name</label>
            <input
                id="Fname"
                type="text"
                bind:value={fname}
                class="border pl-2"
            />
            <label for="Lname">Last name</label>
            <input
                id="Lname"
                type="text"
                bind:value={lname}
                class="border pl-2"
            />
            <button type="submit" class="border" on:click={click}>Submit</button
            >
        </div>
    </div>

    <!-- RIGHT -->
        <div class="flex w-full 5 p-5">
            <div class="flex flex-col w-full">
                <strong class="text-center m-4">User List</strong>

                {#if isLoading}
                    <div class="flex w-full h-full justify-center items-center">
                        <p class="animate-pulse">Loading...</p>
                    </div>
                {:else}
                    <!-- Show user data once loaded -->
                    <ScrollArea class="flex w-full">
                        <div class="grid grid-cols-4 p-4 gap-5">
                            <strong>Firstname</strong>
                            <strong>Lastname</strong>
                        </div>
                        {#each userData as user (user.id)}
                            <div
                                class="grid grid-cols-4 gap-4 p-4 justify-between items-center max-lg:text-[16px]"
                            >
                                <p>{user.fname}</p>
                                <p>{user.lname}</p>
                                <Dialog.Root>
                                    <Dialog.Trigger
                                        class={buttonVariants({
                                            variant: "outline",
                                        })}>Edit</Dialog.Trigger
                                    >
                                    <Dialog.Content class="sm:max-w-[425px]">
                                        <Dialog.Header>
                                            <Dialog.Title>Edit</Dialog.Title>
                                        </Dialog.Header>
                                        <div class="grid gap-4 py-4">
                                            <div
                                                class="grid grid-cols-4 items-center gap-4"
                                            >
                                                <Label
                                                    for="id"
                                                    class="text-right">ID</Label
                                                >
                                                <Input
                                                    id="id_user"
                                                    value={user.id}
                                                    disabled
                                                    class="col-span-3"
                                                />
                                            </div>
                                            <div
                                                class="grid grid-cols-4 items-center gap-4"
                                            >
                                                <Label
                                                    for="name"
                                                    class="text-right"
                                                    >Name</Label
                                                >
                                                <Input
                                                    id="name"
                                                    placeholder={user.fname}
                                                    bind:value={newfname}
                                                    class="col-span-3"
                                                />
                                            </div>
                                            <div
                                                class="grid grid-cols-4 items-center gap-4"
                                            >
                                                <Label
                                                    for="lname"
                                                    class="text-right"
                                                    >Lastname</Label
                                                >
                                                <Input
                                                    id="lname"
                                                    placeholder={user.lname}
                                                    bind:value={newlname}
                                                    class="col-span-3"
                                                />
                                            </div>
                                        </div>
                                        <Dialog.Footer>
                                            <Button
                                                type="submit"
                                                on:click={edit}
                                                >Save changes</Button
                                            >
                                        </Dialog.Footer>
                                    </Dialog.Content>
                                </Dialog.Root>

                                <!-- DELETE -->
                                <Dialog.Root>
                                    <Dialog.Trigger
                                        class={buttonVariants({
                                            variant: "outline",
                                        })}><Trash2 /></Dialog.Trigger
                                    >
                                    <Dialog.Content class="sm:max-w-[425px]">
                                        <Dialog.Header>
                                            <Dialog.Title>Delete</Dialog.Title>
                                            <div class="grid gap-4 py-4">
                                                <div
                                                    class="grid grid-cols-4 items-center gap-4"
                                                >
                                                    <Label
                                                        for="id"
                                                        class="text-right"
                                                        >ID</Label
                                                    >
                                                    <Input
                                                        id="id_user"
                                                        value={user.id}
                                                        disabled
                                                        class="col-span-3"
                                                    />
                                                </div>
                                                <div
                                                    class="grid grid-cols-4 items-center gap-4"
                                                >
                                                    <Label
                                                        for="name"
                                                        class="text-right"
                                                        >Name</Label
                                                    >
                                                    <Input
                                                        id="name"
                                                        value={user.fname}
                                                        disabled
                                                        class="col-span-3"
                                                    />
                                                </div>
                                                <div
                                                    class="grid grid-cols-4 items-center gap-4"
                                                >
                                                    <Label
                                                        for="lname"
                                                        class="text-right"
                                                        >Lastname</Label
                                                    >
                                                    <Input
                                                        id="lname"
                                                        value={user.lname}
                                                        disabled
                                                        class="col-span-3"
                                                    />
                                                </div>
                                            </div>
                                        </Dialog.Header>
                                        <div
                                            class="flex justify-center gap-4 py-4"
                                        >
                                            <Button on:click={deleteuser}
                                                >Delete</Button
                                            >
                                            <Button on:click={() => {}}
                                                >Cancel</Button
                                            >
                                        </div>
                                    </Dialog.Content>
                                </Dialog.Root>
                            </div>
                        {/each}
                    </ScrollArea>
                {/if}
            </div>
        </div>
    </div>

</div>
<footer class="flex w-full h-[50px] bg-[#2f2f2f] justify-center max-lg:mt-32">
    <div class="flex items-center p-3">
        <p class="text-white">
            © 2024 | Made with ❤️ by <a href="https://github.com/tony007x"
                >Tony219</a
            >y
        </p>
    </div>
</footer>
