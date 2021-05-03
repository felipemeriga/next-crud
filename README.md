# Class 14 - Update Customer Hook

### [SWITCH TO PORTUGUESE VERSION](./PT.md)

Now that we have already the update customer component, the only
thing left is the hooks, and the HTTP fetch function, first let's create a PUT
HTTP request, in [services/fetch.ts](services/fetch.ts), add this following method in 
the file that already exists in your project:
```typescript jsx
export async function updateCustomer(customer: ICustomer): Promise<ICustomer> {
    return await getAxiosInstance().put(`/customers/${customer._id}/update`, customer)
}

```

Then, in the file [hooks/hooks.ts](hooks/hooks.ts), add the following function, to the 
existing code:
```typescript jsx
export function useMutationUpdateCustomer(): UseMutationResult<
    ICustomer,
    unknown,
    ICustomer,
    unknown
> {
    return useMutation('customer', (data: ICustomer) => updateCustomer(data))
}

```

Also, let's create the hook for intercepting the PUT HTTP request, so in the 
[services/mock.ts](services/mock.ts) file, add this part to the existing code:

```typescript jsx
mock.onPut(url).reply(function (config) {
    if (config.url) {
        const id = String(config.url.match(/\d/g))

        const index: number | undefined = sampleCustomerData.findIndex(
            (value: ICustomer) => value._id === id
        )
        const data: ICustomer = JSON.parse(config.data)
        data._id = id
        sampleCustomerData[index] = data
        return [200, data]
    } else {
        return [500, 'response']
    }
})

```

Now, that we have the hook and the HTTP method, the entire update page [/pages/customers/[id]/update.tsx](/pages/customers/[id]/update.tsx),
will be like:
```typescript jsx
import Layout from '../../../components/Layout'
import { useRouter } from 'next/router'
import { useCustomer, useMutationUpdateCustomer } from '../../../hooks/hooks'
import { ICustomer } from '../../../interfaces'
import React, { useState } from 'react'
import Loading from '../../../components/Loading'
import Form from '../../../components/Form'

function Update(): JSX.Element {
    const [errorMessage, setErrorMessage] = useState('')
    const mutation = useMutationUpdateCustomer()

    const router = useRouter()
    const { id } = router.query

    const { data, error } = useCustomer(String(id))
    const customer: ICustomer = data as ICustomer

    if (error) return <div>failed to load</div>

    const onSubmitCallback = (formData: ICustomer) => {
        if (errorMessage) setErrorMessage('')
        formData._id = String(id)
        mutation.mutate(formData)
    }

    if (mutation.error) {
        setErrorMessage('Error updating the user')
    }

    if (mutation.isSuccess) {
        router.push('/')
    }

    return (
        <Layout>
            {customer ? (
                <div>
                    <h1>Edit Customer</h1>
                    <Form submitFormCallback={onSubmitCallback} defaultValues={customer} />
                    {errorMessage && (
                        <p role="alert" className="errorMessage">
                            {errorMessage}
                        </p>
                    )}
                </div>
            ) : (
                <Loading />
            )}
        </Layout>
    )
}

export default Update

```